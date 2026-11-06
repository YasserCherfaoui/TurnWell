# 0007 — Testing & Verification Strategy

## 1. Context & Motivation

Module [`M01-auth-accounts.md`](../../turnwell-specs/modules/M01-auth-accounts.md) (section 14) requires:
- **Unit Tests:** Password policy, Argon2 verify, HMAC email hash normalization, refresh rotation, reuse detection revoking family.
- **E2E Tests:** Sign-up $\rightarrow$ verify $\rightarrow$ `/me`, wrong code $\times 5 \rightarrow$ locked, endpoint without age $\rightarrow$ `AGE_NOT_CONFIRMED`, forgot password for unknown email returns 200.

Testing runs with **Vitest** for unit tests and **Supertest** for end-to-end integration tests.

---

## 2. Unit Test Suite (Vitest)

Unit tests run in seconds without external network or database dependencies:

| Test File | Focus Areas | Tests |
| :--- | :--- | :--- |
| [`crypto.service.spec.ts`](../../server/src/modules/auth/crypto.service.spec.ts) | Argon2id `$argon2id$v=19$m=19456,t=2,p=1$`, HMAC email normalization, AES-256-GCM encryption/decryption | 3 |
| [`token.service.spec.ts`](../../server/src/modules/auth/token.service.spec.ts) | Ed25519 signing, claims (`sub`, `role`, `age`, `iss`, `aud`), 15 min expiry, rejection of malformed tokens | 3 |
| [`code-store.service.spec.ts`](../../server/src/modules/auth/code-store.service.spec.ts) | 6-digit code generation/consumption, 60s cooldown limit, max 5 attempts lockout (`AUTH_CODE_LOCKED`) | 3 |
| [`auth.service.spec.ts`](../../server/src/modules/auth/auth.service.spec.ts) | Sign-up with 18+ check, email duplication prevention, refresh token rotation, replay attack family revocation | 5 |
| [`users.service.spec.ts`](../../server/src/modules/users/users.service.spec.ts) | Status flags & entitlements computation, 18+ age confirmation, under-18 soft-delete & session revocation | 3 |

**Total Unit Tests:** 17 tests passed.

---

## 3. End-to-End Test Suite (Supertest)

### Mocking the Database with `TestPrismaService`
File: [`server/test/test-prisma.service.ts`](../../server/test/test-prisma.service.ts)

To run reliable E2E tests without needing a live Cloud SQL / local PostgreSQL instance, `TestPrismaService` implements an in-memory replica of Prisma's `user`, `refreshToken`, `device`, `profile`, and `keyBackup` delegates.

In [`server/test/app.e2e-spec.ts`](../../server/test/app.e2e-spec.ts), NestJS overrides the real `PrismaService`:
```typescript
const moduleFixture = await Test.createTestingModule({
  imports: [AppModule],
})
  .overrideProvider(PrismaService)
  .useValue(new TestPrismaService())
  .compile();

app = moduleFixture.createNestApplication();
app.setGlobalPrefix('v1', { exclude: ['health'] });
app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }));
app.useGlobalFilters(new GlobalExceptionFilter());
await app.init();
```

### E2E Test Cases Verified
1. **Health Check:** `GET /health` returns 200 with `{ status: "ok", version: "1.0.0" }`.
2. **18+ Enforcement at Sign-Up:** `POST /v1/auth/sign-up` with `ageConfirmed: false` returns HTTP 403 `AGE_NOT_CONFIRMED`.
3. **Password Validation:** `POST /v1/auth/sign-up` with `password < 10` chars returns HTTP 400 `VALIDATION_FAILED`.
4. **Happy Path Flow:**
   - Sign up $\rightarrow$ HTTP 201 `{ userId, next: "VERIFY_EMAIL" }`.
   - Intercept code via `MailService.lastSentCode`.
   - Submit wrong code $\rightarrow$ HTTP 400 `AUTH_CODE_INVALID` with `attemptsLeft: 4`.
   - Submit correct code $\rightarrow$ HTTP 200 with `accessToken` + `refreshToken`.
   - Call protected `GET /v1/me` with Bearer token $\rightarrow$ HTTP 200.
   - Call `POST /v1/auth/refresh` $\rightarrow$ HTTP 200 with rotated tokens.
   - Replay old refresh token $\rightarrow$ HTTP 401 `AUTH_REFRESH_REUSED`.
   - Call `POST /v1/auth/sign-out` $\rightarrow$ HTTP 200.
5. **Anti-Enumeration:** `POST /v1/auth/password/forgot` returns HTTP 200 for non-existent email without error.
6. **Unauthorized Access:** `GET /v1/me` without Bearer token returns HTTP 401 `AUTH_INVALID`.

**Total E2E Tests:** 6 tests passed.

---

## 4. Verification Commands

From the `server/` directory:

```bash
# Run unit tests
npm test

# Run e2e tests
npm run test:e2e

# Run linter (OxLint)
npm run lint

# Run build
npm run build
```

All 23 tests pass cleanly and the linter reports 0 errors and 0 warnings.

