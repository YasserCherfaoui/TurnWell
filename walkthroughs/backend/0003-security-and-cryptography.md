# 0003 — Security & Cryptography

## 1. Context & Motivation

TurnWell handles sensitive personal health and spiritual recovery data.
Module [`M01-auth-accounts.md`](../../turnwell-specs/modules/M01-auth-accounts.md) and [`AGENTS.md`](../../turnwell-specs/AGENTS.md) establish non-negotiable security requirements:
- Passwords must be hashed with memory-hard **Argon2id**.
- Emails must not be stored in plaintext. Lookups use a keyed **HMAC-SHA256** hash, and the email itself is encrypted via **AES-256-GCM**.
- Access tokens must be short-lived (15 min) and signed using modern public-key **Ed25519 (EdDSA)**.
- Refresh tokens must rotate and detect **token replay attacks**, revoking the entire token family if compromised.

All cryptography is implemented in [`CryptoService`](../../server/src/modules/auth/crypto.service.ts) and [`TokenService`](../../server/src/modules/auth/token.service.ts).

---

## 2. Argon2id Password Hashing

### Parameter Requirements
As defined in `M01-auth-accounts.md` (section 7):
- Algorithm: `argon2id` (Argon2 version 19).
- Memory cost: `m = 19456` (19 MiB per hash to prevent ASIC/GPU parallel cracking).
- Time cost: `t = 2` iterations.
- Parallelism: `p = 1` thread.

### Implementation: `server/src/modules/auth/crypto.service.ts`
```typescript
import * as argon2 from '@node-rs/argon2';

export class CryptoService {
  async hashPassword(password: string): Promise<string> {
    return argon2.hash(password, {
      memoryCost: 19456,
      timeCost: 2,
      parallelism: 1,
      algorithm: 2, // 2 = Argon2id
    });
  }

  async verifyPassword(hash: string, password: string): Promise<boolean> {
    try {
      return await argon2.verify(hash, password);
    } catch {
      return false;
    }
  }
}
```
Resulting PHC string format:
`$argon2id$v=19$m=19456,t=2,p=1$...`

---

## 3. Zero-Knowledge Email: HMAC-SHA256 & AES-256-GCM

To prevent database leaks from exposing user email addresses:

1. **`User.emailHash` (Lookup Key):**
   - The email is trimmed and converted to lowercase: `email.trim().toLowerCase()`.
   - Computed with a secret HMAC key:
     ```typescript
     hashEmail(email: string): string {
       const normalized = email.trim().toLowerCase();
       return hmacSha256(this.emailHashSecret, normalized);
     }
     ```
   - Attackers with database dumps cannot look up users by plain SHA-256 rainbow tables without `EMAIL_HASH_SECRET`.

2. **`User.emailEnc` (Encrypted Storage):**
   - Encrypted with AES-256-GCM envelope encryption:
     ```typescript
     export function encryptAesGcm(plainText: string, key32Bytes: Buffer): string {
       const iv = randomBytes(12);
       const cipher = createCipheriv('aes-256-gcm', key32Bytes, iv);
       const encrypted = Buffer.concat([cipher.update(plainText, 'utf8'), cipher.final()]);
       const tag = cipher.getAuthTag();
       return `${iv.toString('base64url')}.${tag.toString('base64url')}.${encrypted.toString('base64url')}`;
     }
     ```
   - Only decrypted when sending verification or reset emails.

---

## 4. Ed25519 (EdDSA) JWT Access Tokens

Access tokens are signed using the modern **Ed25519** elliptic curve algorithm via the `jose` library.

### Implementation: `server/src/modules/auth/token.service.ts`
```typescript
@Injectable()
export class TokenService implements OnModuleInit {
  private privateKey!: CryptoKey | Uint8Array;
  private publicKey!: CryptoKey | Uint8Array;
  private readonly kid = 'turnwell-key-1';
  private readonly issuer = 'turnwell-api';
  private readonly audience = 'turnwell-app';
  private readonly accessExpirySeconds = 900; // 15 minutes

  async onModuleInit() {
    const rawPrivateKey = this.config.get<string>('JWT_PRIVATE_KEY');
    const rawPublicKey = this.config.get<string>('JWT_PUBLIC_KEY');

    if (rawPrivateKey && rawPublicKey) {
      this.privateKey = (await jose.importPKCS8(rawPrivateKey, 'EdDSA')) as CryptoKey;
      this.publicKey = (await jose.importSPKI(rawPublicKey, 'EdDSA')) as CryptoKey;
    } else {
      // In development / testing, generate an in-memory Ed25519 keypair
      const keyPair = await jose.generateKeyPair('EdDSA', { extractable: true });
      this.privateKey = keyPair.privateKey;
      this.publicKey = keyPair.publicKey;
    }
  }

  async issuePair(user: { id: string; role: string; ageConfirmedAt?: Date | null }): Promise<TokenPair> {
    const ageConfirmed = !!user.ageConfirmedAt;
    const now = Math.floor(Date.now() / 1000);

    const accessToken = await new jose.SignJWT({ role: user.role, age: ageConfirmed })
      .setProtectedHeader({ alg: 'EdDSA', kid: this.kid })
      .setSubject(user.id)
      .setIssuer(this.issuer)
      .setAudience(this.audience)
      .setIssuedAt(now)
      .setExpirationTime(now + this.accessExpirySeconds)
      .sign(this.privateKey);

    const refreshToken = generateRandomToken(32);
    return { accessToken, refreshToken, expiresIn: this.accessExpirySeconds };
  }
}
```

---

## 5. Refresh Token Rotation & Family Reuse Detection

To protect mobile devices from token theft:

1. When a user authenticates, a random 32-byte base64url string is generated.
2. The database stores only `sha256(refreshToken)` under a unique `familyId`.
3. When `/v1/auth/refresh` is called:
   - If the token was **already revoked** (e.g. an attacker intercepted an old token that the app already refreshed), it indicates a replay attack!
   - The backend immediately executes:
     ```typescript
     await this.prisma.refreshToken.updateMany({
       where: { familyId: tokenRecord.familyId },
       data: { revokedAt: new Date() },
     });
     throw new AppException({
       code: ErrorCodes.AUTH_REFRESH_REUSED,
       message: 'Revoked refresh token reused. All sessions in this family have been terminated.',
       status: 401,
     });
     ```
   - If valid: the old token is revoked, a new token is created under the **same `familyId`**, and the new pair is returned.

---

## 6. Anti-Enumeration & Constant-Time Protection

1. **`POST /v1/auth/password/forgot`:**
   Returns HTTP 200 with `{ message: "If an account exists, a password reset code has been sent." }` regardless of whether the email is registered or not.
2. **`POST /v1/auth/sign-in`:**
   If the email does not exist, a dummy Argon2id verification is run against a static hash so response times do not reveal account existence.

