# 0006 — Auth & Users Modules Implementation

## 1. Context & Motivation

This chapter details the controller, service, and DTO layers for:
- **`AuthModule`** ([`server/src/modules/auth/`](../../server/src/modules/auth/)): Account creation, login, email verification, OAuth, refresh tokens, and password reset.
- **`UsersModule`** ([`server/src/modules/users/`](../../server/src/modules/users/)): Profile inspection (`/v1/me`), age confirmation gate, and device/push token management.
- **`HealthModule`** ([`server/src/modules/health/`](../../server/src/modules/health/)): Root health check (`/health`).

---

## 2. API Endpoints Breakdown

### Auth Module Endpoints

```
POST /v1/auth/sign-up
POST /v1/auth/verify-email
POST /v1/auth/resend-code
POST /v1/auth/sign-in
POST /v1/auth/oauth
POST /v1/auth/refresh
POST /v1/auth/sign-out
POST /v1/auth/password/forgot
POST /v1/auth/password/reset
```

#### 1. Sign-Up: `POST /v1/auth/sign-up`
- **Request DTO:**
  ```json
  {
    "email": "user@example.com",
    "password": "correct-horse-42",
    "ageConfirmed": true,
    "device": { "platform": "IOS", "appVersion": "1.0.0" }
  }
  ```
- **Validation:**
  - `email`: Valid email format.
  - `password`: String $\ge 10$ characters.
  - `ageConfirmed`: Must be `true`. If `false`, throws `AGE_NOT_CONFIRMED` (403).
- **Execution:**
  1. Computes `emailHash = HMAC(email)`.
  2. Rejects with `AUTH_EMAIL_TAKEN` (409) if user exists.
  3. Hashes password via Argon2id. Encrypts email via AES-256-GCM.
  4. Generates UUIDv7 `userId` and inserts record into `account.users`.
  5. Generates 6-digit code, saves in `CodeStoreService`, and sends via `MailService`.
  6. Returns `201 Created`: `{ "userId": "...", "next": "VERIFY_EMAIL" }`.

#### 2. Verify Email: `POST /v1/auth/verify-email`
- **Request:** `{ "email": "...", "code": "123456" }`
- **Execution:**
  1. Verifies code with `CodeStoreService.verifyCode('VERIFY', emailHash, code)`.
  2. Sets `User.emailVerifiedAt = now()`.
  3. Creates an initial session (issues Ed25519 access token + 30-day refresh token).
  4. Returns:
     ```json
     {
       "accessToken": "eyJ...",
       "refreshToken": "...",
       "expiresIn": 900,
       "user": {
         "id": "...",
         "ageConfirmed": true,
         "onboardingDone": false,
         "hasKeyBackup": false,
         "role": "USER"
       }
     }
     ```

#### 3. Sign-In: `POST /v1/auth/sign-in`
- **Request:** `{ "email": "...", "password": "..." }`
- **Execution:**
  1. Finds user by `emailHash`.
  2. If missing or provider is not `EMAIL`: executes constant-time dummy Argon2 verification and throws `AUTH_INVALID` (401).
  3. Verifies password hash with `argon2.verify()`.
  4. Checks `emailVerifiedAt != null`. If not verified, throws `AUTH_INVALID` with `{ emailVerified: false }`.
  5. Updates user timezone if changed.
  6. Creates and returns session token pair.

#### 4. Refresh Tokens: `POST /v1/auth/refresh`
- **Request:** `{ "refreshToken": "..." }`
- **Execution:**
  1. Finds record by `sha256(refreshToken)`.
  2. If already revoked $\rightarrow$ **Replay attack detected**: revokes entire `familyId`, throws `AUTH_REFRESH_REUSED` (401).
  3. If valid: marks old token revoked, generates new refresh token with the **same `familyId`**, signs new access token, and returns the pair.

---

### Users Module Endpoints

```
GET  /v1/me
POST /v1/me/age-confirmation
PUT  /v1/me/device
POST /v1/devices/current/push-token
```

#### 1. Current User State: `GET /v1/me`
- Requires Bearer auth + age confirmed.
- Returns comprehensive client status:
  ```json
  {
    "id": "0193...",
    "role": "USER",
    "locale": "ar",
    "timezone": "Africa/Algiers",
    "ageConfirmed": true,
    "emailVerified": true,
    "onboardingDone": false,
    "hasKeyBackup": false,
    "deletionScheduledAt": null,
    "entitlements": {
      "premium": false,
      "aiDailyLimit": 15
    }
  }
  ```
  *(Note: Email is never returned here for privacy; settings displays masked email from a separate dedicated route).*

#### 2. Age Confirmation Gate: `POST /v1/me/age-confirmation`
- Marked with `@AllowWithoutAge()` so users without `ageConfirmedAt` can call it.
- **If `confirmed: true`:** Sets `User.ageConfirmedAt = now()`.
- **If `confirmed: false`:** User declared they are under 18. Per US-01.4, account is immediately soft-deleted (`deletedAt = now()`) and all refresh sessions are revoked.

#### 3. Device & Push Token: `POST /devices/current/push-token`
- Upserts the device row with the client FCM/APNs push token and returns `204 No Content`.

---

### Health Module: `GET /health`

- Marked with `@Public()`.
- Excluded from the global `/v1` prefix so Cloud Run / Kubernetes readiness probes query `http://host:port/health` directly.
- Returns `200 OK`: `{ "status": "ok", "version": "1.0.0", "timestamp": "..." }`.

