# 02 — Conventions

Shared rules for every module. Module specs reference these instead of repeating them.

## 1. Repository layout

```
turnwell/
├── apps/
│   ├── mobile/                      # Flutter
│   │   ├── lib/
│   │   │   ├── main.dart
│   │   │   ├── app.dart             # MaterialApp.router, theme, l10n
│   │   │   ├── core/
│   │   │   │   ├── theme/           # tokens, GlassSurface, MeshBackground (see 04-design-system.md)
│   │   │   │   ├── crypto/          # KeyManager, Cipher, Argon2 (M06)
│   │   │   │   ├── db/              # Drift database, DAOs (M06)
│   │   │   │   ├── network/         # Dio, interceptors, generated API client
│   │   │   │   ├── l10n/            # ARB files (M16)
│   │   │   │   ├── router/          # go_router config + guards
│   │   │   │   └── widgets/         # shared widgets
│   │   │   └── features/
│   │   │       └── <feature>/
│   │   │           ├── data/        # repositories, DTO mappers, local DAOs
│   │   │           ├── domain/      # entities (freezed), value objects
│   │   │           └── presentation/# screens, widgets, providers
│   │   └── test/
│   ├── api/                         # NestJS
│   │   ├── src/
│   │   │   ├── main.ts
│   │   │   ├── app.module.ts
│   │   │   ├── common/              # guards, interceptors, filters, decorators, redaction
│   │   │   ├── prisma/              # PrismaService
│   │   │   └── modules/<module>/
│   │   │       ├── <module>.module.ts
│   │   │       ├── <module>.controller.ts
│   │   │       ├── <module>.service.ts
│   │   │       ├── dto/
│   │   │       ├── jobs/            # BullMQ processors (if any)
│   │   │       └── <module>.spec.ts
│   │   ├── prisma/schema.prisma     # copy of schema/schema.prisma
│   │   └── test/                    # e2e
│   └── admin/                       # Moderator console (Next.js) — M13
├── packages/
│   ├── prompts/                     # copy of prompts/ — versioned
│   └── contracts/                   # openapi.json → generated Dart + TS clients
└── infra/                           # Terraform (M18)
```

## 2. Naming

| Thing | Convention | Example |
| --- | --- | --- |
| Dart files | `snake_case.dart` | `check_in_screen.dart` |
| Dart classes | `PascalCase` | `CheckInScreen` |
| Riverpod providers | `camelCase` + `Provider` suffix (generated) | `todayCheckInsProvider` |
| Routes | kebab-case paths | `/tracking/check-in` |
| NestJS files | `kebab-case.<type>.ts` | `coach-message.dto.ts` |
| DB tables | `snake_case`, plural | `tracked_items` |
| Prisma models | `PascalCase`, singular | `TrackedItem` |
| Enums | `SCREAMING_SNAKE` values | `SLIP`, `CHECK_IN` |
| API JSON fields | `camelCase` | `occurredOn` |
| Feature IDs | `F-xx` (from `00-product-overview.md`) | `F-31` |
| Error codes | `MODULE_REASON` | `COACH_DAILY_LIMIT` |
| Analytics events | `module.object_action` | `sos.flow_started` |

## 3. API conventions

- Base URL: `https://api.turnwell.app/v1` (staging: `api-staging.turnwell.app`).
- Auth: `Authorization: Bearer <accessJwt>`. Access token 15 min, refresh token 30 days (rotating). See M01.
- Headers sent by the app: `X-App-Version`, `X-Platform` (`ios`|`android`), `Accept-Language`, `X-Timezone` (IANA).
- IDs: UUIDv7 strings. Dates: ISO 8601 UTC (`2026-11-02T21:30:00Z`). Calendar days: `YYYY-MM-DD` in the user's timezone.
- Pagination: cursor-based. `?cursor=<opaque>&limit=20` → `{ "items": [...], "nextCursor": "..." | null }`.
- Encrypted payloads are sent as base64url strings in a field named `payloadEnc` with `encVersion` (int).
- Streaming (coach replies): Server-Sent Events, `text/event-stream`, events `token`, `safety`, `done`, `error`.
- Realtime (community): Socket.IO namespace `/community`, auth via `auth.token`.

### Error format (all endpoints)

```json
{
  "error": {
    "code": "COACH_DAILY_LIMIT",
    "message": "Daily message limit reached.",
    "details": { "limit": 15, "resetsAt": "2026-11-03T00:00:00+01:00" },
    "requestId": "01J…"
  }
}
```

`message` is developer-facing English. The app maps `code` → localized text. Never put user content in errors.

### Common error codes

| HTTP | Code | When |
| --- | --- | --- |
| 400 | `VALIDATION_FAILED` | DTO validation failed (`details.fields`) |
| 401 | `AUTH_TOKEN_EXPIRED` / `AUTH_INVALID` | JWT problems |
| 403 | `FORBIDDEN` | Role / ownership |
| 403 | `AGE_NOT_CONFIRMED` | 18+ gate not passed |
| 404 | `NOT_FOUND` | Resource missing or not owned (never reveal which) |
| 409 | `CONFLICT` | Sync version conflict |
| 423 | `COMMUNITY_RESTRICTED` | Muted / banned pseudonym |
| 429 | `RATE_LIMITED` | Throttler (`details.retryAfter`) |
| 503 | `AI_UNAVAILABLE` | LLM provider down (app shows offline coach card) |

Module-specific codes are listed in each module's API section, e.g. `AUTH_EMAIL_TAKEN`, `AUTH_CODE_INVALID`,
`AUTH_CODE_EXPIRED`, `AUTH_CODE_LOCKED`, `AUTH_OAUTH_INVALID`, `AUTH_PROVIDER_MISMATCH`, `AUTH_REFRESH_REUSED`,
`AUTH_REAUTH_REQUIRED` (M01) · `ONBOARDING_INVALID_COACH_NAME`, `ONBOARDING_ALREADY_DONE`, `ONBOARDING_NOT_ENOUGH_INFO`,
`PLAN_GENERATION_FAILED` (M03) · `SYNC_BATCH_TOO_LARGE`, `ENTRY_TOO_LARGE` (M06) · `COACH_DAILY_LIMIT` (M07).

## 4. Backend rules

- `ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true })` globally.
- `@nestjs/throttler` defaults: 60 req/min per user, 20 req/min per IP for unauthenticated routes. Module specs may tighten.
- Logging: `pino` with a redaction list (`req.body`, `*.content`, `*.payloadEnc`, `*.text`, `*.body`, `authorization`).
- Every service method that touches another user's data checks ownership; return `NOT_FOUND`, not `FORBIDDEN`.
- Background jobs: BullMQ queues named `<module>.<job>` (e.g. `nudges.dispatch`). Idempotent processors.
- Config via `@nestjs/config` + zod schema; secrets from Secret Manager at boot.
- Roles: `USER`, `MODERATOR`, `ADMIN` (`@Roles()` decorator + `RolesGuard`).

## 5. Flutter rules

- Riverpod `@riverpod` generated providers; `AsyncValue` for async UI; no `setState` business logic.
- Entities with `freezed` + `json_serializable`.
- Repositories expose `Stream`s from Drift for offline-first features; network calls are background sync.
- Every screen: loading, empty, error and offline states.
- Navigation guards in `core/router`: `AuthGuard`, `LockGuard` (app lock), `OnboardingGuard`.
- Haptics via `HapticFeedback` only on positive actions.
- Accessibility: every tappable has a `Semantics` label; min target 48×48.

## 6. Testing

| Layer | Tool | Minimum |
| --- | --- | --- |
| Backend unit | Jest | Every service public method |
| Backend e2e | Jest + Supertest + Testcontainers (Postgres, Redis) | Every endpoint happy path + auth + 1 error |
| Prompts | `packages/prompts/tests/*.yaml` run by `pnpm prompt:test` | Every safety level, each mode, each language |
| Flutter unit | `flutter_test` | Repositories, crypto, providers |
| Flutter widget | `flutter_test` + golden tests | Each screen in LTR + RTL, light + dark |
| Integration | `integration_test` | Onboarding, check-in, SOS, post |

## 7. Module spec template

Every file in `modules/` follows this structure (sections may say "N/A"):

1. **Header** — ID, name, release, feature IDs, owners (backend module / Flutter feature folder), dependencies
2. **Purpose** — one paragraph
3. **User stories & acceptance criteria** — `Given / When / Then`
4. **Screens** — inventory table + ASCII wireframes + component notes
5. **Flows** — Mermaid flowchart / sequence diagrams
6. **State machine** — Mermaid `stateDiagram-v2` where relevant
7. **Data** — tables used (link to `03-data-model.md`), local Drift tables, field rules
8. **API** — endpoints table + request/response JSON examples + error codes
9. **Backend design** — services, methods, jobs, guards
10. **Flutter design** — folder tree, providers, routes, key widgets
11. **Edge cases**
12. **Security & privacy**
13. **Analytics events** — names + properties (never content)
14. **Tests** — must-have cases
15. **Open questions**
