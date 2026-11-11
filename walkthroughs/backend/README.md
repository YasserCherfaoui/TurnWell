# Backend Walkthroughs

Step-by-step educational documentation of the TurnWell NestJS backend foundation and module implementations.

## Chapters

| # | Chapter | Key Topics Covered |
| :--- | :--- | :--- |
| **0001** | [0001 — Setup & Prisma Multi-Schema ORM](./0001-setup-and-prisma-orm.md) | Project initialization, dependencies, Prisma 5 multi-schema configuration, PostgreSQL schema architecture (`account`, `private`, `community`, `system`). |
| **0002** | [0002 — Common Foundation & Privacy Layer](./0002-common-foundation-and-privacy.md) | UUIDv7 time-ordered IDs, standard `{ error: { code, message, details, requestId } }` format, `GlobalExceptionFilter`, standard error codes. |
| **0003** | [0003 — Security & Cryptography](./0003-security-and-cryptography.md) | Argon2id password hashing (`m=19456, t=2, p=1`), HMAC-SHA256 email hashing, AES-256-GCM envelope encryption, Ed25519 (EdDSA) access JWTs, refresh token rotation with family reuse/replay detection. |
| **0004** | [0004 — Guards & Decorators Pipeline](./0004-guards-and-decorators.md) | Request lifecycle, `JwtAuthGuard` (`@Public()`), `AgeConfirmedGuard` (`@AllowWithoutAge()`), `RolesGuard` (`@Roles()`), `@CurrentUser()`. |
| **0005** | [0005 — Code Store & Email Verification Engine](./0005-code-store-and-verification.md) | Redis & in-memory verification code store, 15-min TTL, 60s cooldown limit, max 5 attempts lockout (`AUTH_CODE_LOCKED`). |
| **0006** | [0006 — Auth & Users Modules Implementation](./0006-auth-and-users-modules.md) | Implementation of all M01 auth endpoints, `/v1/me`, 18+ gate confirmation, device/push tokens, and `/health`. |
| **0007** | [0007 — Testing & Verification Strategy](./0007-testing-and-verification.md) | Vitest unit tests, `TestPrismaService` in-memory mock, Supertest E2E integration test suite, linter, and build commands. |
| **0008** | [0008 — Backend Environment & Configuration](./0008-backend-environment-and-configuration.md) | `.env.example` setup, configuration schema, zero-friction local dev fallbacks (in-memory JWT & OTP store), and quick-start guide. |
| **0009** | [0009 — Onboarding Setup & Profile Configuration (M03)](./0009-onboarding-setup-and-profile.md) | `OnboardingModule`, `PUT /v1/onboarding/setup`, `GET /v1/onboarding/state`, 4-language authority title blocklist validator (`ONBOARDING_INVALID_COACH_NAME`), `private.profiles` upsert, and `MeDto.onboardingStep`. |


