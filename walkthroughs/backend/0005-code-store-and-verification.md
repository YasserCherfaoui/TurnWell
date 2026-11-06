# 0005 — Code Store & Email Verification Engine

## 1. Context & Motivation

Module [`M01-auth-accounts.md`](../../turnwell-specs/modules/M01-auth-accounts.md) specifies a 6-digit email code flow for:
- Email verification at sign-up.
- Password reset authorization.

Strict security constraints apply:
1. **Expiry:** Valid for 15 minutes.
2. **Cooldown:** Resending has a 60-second cooldown to prevent mail flooding.
3. **Brute Force Lockout:** Max 5 incorrect attempts. On the 5th failure, the code is invalidated (`AUTH_CODE_LOCKED`).
4. **Zero-Knowledge Storage:** Verification codes are never stored in PostgreSQL; they reside in Redis (or in-memory fallback) under key `auth:code:<purpose>:<emailHash>`.

All code store logic is implemented in [`server/src/modules/auth/code-store.service.ts`](../../server/src/modules/auth/code-store.service.ts).

---

## 2. Data Structure

The code record is stored as:
```typescript
export interface CodeData {
  codeHash: string; // sha256(plainCode)
  attempts: number; // 0 to 5
  createdAt: number; // ms timestamp for cooldown check
}
```

Key format:
`auth:code:<purpose>:<emailHash>`
- `purpose`: `'VERIFY'` | `'RESET'`
- `emailHash`: HMAC-SHA256 hex string of the normalized email.

---

## 3. Cooldown & Save Logic

When `saveCode` is called:
```typescript
async saveCode(purpose: string, emailHash: string, plainCode: string): Promise<void> {
  const key = this.getKey(purpose, emailHash);
  const now = Date.now();

  // Check 60s cooldown
  const existing = await this.getCode(purpose, emailHash);
  if (existing && now - existing.createdAt < this.cooldownSeconds * 1000) {
    const waitTime = Math.ceil((this.cooldownSeconds * 1000 - (now - existing.createdAt)) / 1000);
    throw new AppException({
      code: ErrorCodes.AUTH_CODE_COOLDOWN,
      message: `Please wait ${waitTime} seconds before requesting a new code.`,
      details: { retryAfterSeconds: waitTime },
      status: 429,
    });
  }

  const data: CodeData = {
    codeHash: sha256(plainCode),
    attempts: 0,
    createdAt: now,
  };

  if (this.redis) {
    await this.redis.set(key, JSON.stringify(data), 'EX', this.ttlSeconds); // 900s
  } else {
    this.memoryStore.set(key, { data, expiresAt: now + this.ttlSeconds * 1000 });
  }
}
```

---

## 4. Verification & Lockout Logic

When `verifyCode` is called:
```typescript
async verifyCode(purpose: string, emailHash: string, plainCode: string): Promise<boolean> {
  const key = this.getKey(purpose, emailHash);
  const existing = await this.getCode(purpose, emailHash);

  if (!existing) {
    throw new AppException({
      code: ErrorCodes.AUTH_CODE_EXPIRED,
      message: 'Verification code expired or not found.',
      status: 400,
    });
  }

  // Check lockout
  if (existing.attempts >= this.maxAttempts) {
    await this.deleteCode(purpose, emailHash);
    throw new AppException({
      code: ErrorCodes.AUTH_CODE_LOCKED,
      message: 'Too many incorrect attempts. Code is locked.',
      status: 400,
    });
  }

  const inputHash = sha256(plainCode);
  if (inputHash !== existing.codeHash) {
    existing.attempts += 1;
    const attemptsLeft = this.maxAttempts - existing.attempts;

    if (existing.attempts >= this.maxAttempts) {
      await this.deleteCode(purpose, emailHash);
      throw new AppException({
        code: ErrorCodes.AUTH_CODE_LOCKED,
        message: 'Too many incorrect attempts. Code is locked.',
        details: { attemptsLeft: 0 },
        status: 400,
      });
    }

    // Persist incremented attempts
    await this.persistUpdatedAttempts(key, existing);

    throw new AppException({
      code: ErrorCodes.AUTH_CODE_INVALID,
      message: 'Invalid verification code.',
      details: { attemptsLeft },
      status: 400,
    });
  }

  // Success - single use, consume code
  await this.deleteCode(purpose, emailHash);
  return true;
}
```

---

## 5. Dual Engine: Redis + In-Memory Fallback

- **Production / Staging:** Connects to Memorystore Redis via `REDIS_URL`.
- **Local Dev / Testing:** If Redis is not configured or fails to connect, `CodeStoreService` falls back to an in-memory `Map` with identical TTL and attempt semantics, enabling local testing and rapid CI without external services.

