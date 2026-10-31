# 01 — Architecture

TurnWell is one Flutter app + one NestJS modular monolith on Cloud Run + managed GCP services.
Private data is encrypted on the device; the API stores ciphertext, community data and nudge metadata.

## 1. System diagram

```mermaid
flowchart TB
    subgraph Clients
        APP["Flutter app (iOS + Android)<br/>Drift + SQLCipher, offline-first"]
        ADMIN["Moderator console (Next.js)"]
    end

    subgraph API["NestJS API on Cloud Run"]
        AUTH[auth]
        USERS[users]
        SYNC[sync]
        COACH[coach]
        LLM[llm]
        SAFETY[safety]
        NUDGES[nudges]
        COMM[community]
        MOD[moderation]
        BILL["billing (v2)"]
        ADM[admin]
    end

    subgraph GCP["Managed services (europe-west1)"]
        PG[(Cloud SQL PostgreSQL 16)]
        REDIS[(Memorystore Redis<br/>BullMQ, rate limits)]
        KMS[Cloud KMS]
        GCS[(Cloud Storage)]
        VERTEX[Vertex AI — Gemini]
        SCHED[Cloud Scheduler]
        SM[Secret Manager]
    end

    PUSH[FCM / APNs]

    APP -- "HTTPS (REST + SSE)" --> API
    APP -- "Socket.IO /community" --> COMM
    ADMIN -- "HTTPS /mod, /admin" --> API
    API --> PG
    API --> REDIS
    API --> KMS
    API --> GCS
    LLM --> VERTEX
    SCHED -- "every 15 min" --> NUDGES
    NUDGES --> PUSH
    PUSH --> APP
    API --> SM
```

## 2. Module map

| ID | Module | Backend module | Flutter feature | Release |
| --- | --- | --- | --- | --- |
| M01 | Auth & accounts | `auth`, `users` | `auth` | v1 |
| M02 | App lock & disguise | — | `lock` | v1 |
| M03 | Onboarding & plan | `coach` (onboarding service) | `onboarding` | v1 |
| M04 | Tracking | `sync` (metadata) | `tracking` | v1 |
| M05 | Journal | `sync` | `journal` | v1 |
| M06 | Encryption & sync | `sync` | `core/crypto`, `core/db`, `sync` | v1 |
| M07 | AI coach | `coach` | `coach` | v1 |
| M08 | LLM gateway | `llm` | — | v1 |
| M09 | Safety & crisis | `safety` | `safety` | v1 |
| M10 | SOS | `coach` (SOS conversation) | `sos` | v1 |
| M11 | Nudges & notifications | `nudges` | `notifications` | v1 |
| M12 | Community | `community` | `community` | v1 (posts) / v2 (circles, DMs) |
| M13 | Moderation & console | `moderation`, `admin` | `apps/admin` | v1 basic |
| M14 | Settings & personalization | `users` | `settings` | v1 |
| M15 | Subscriptions | `billing` | `paywall` | v2 |
| M16 | Localization | — | `core/l10n` | v1 |
| M17 | Privacy: export, delete, panic wipe | `users` | `privacy` | v1 |
| M18 | Infrastructure & DevOps | — | — | v1 |

## 3. Module dependencies

```mermaid
flowchart LR
    M01[M01 Auth] --> M06[M06 Encryption & sync]
    M02[M02 Lock] --> M06
    M03[M03 Onboarding] --> M07[M07 Coach]
    M03 --> M04[M04 Tracking]
    M04 --> M06
    M05[M05 Journal] --> M06
    M07 --> M08[M08 LLM gateway]
    M07 --> M09[M09 Safety]
    M10[M10 SOS] --> M07
    M10 --> M09
    M11[M11 Nudges] --> M04
    M11 --> M08
    M12[M12 Community] --> M13[M13 Moderation]
    M13 --> M08
    M12 --> M09
    M14[M14 Settings] --> M07
    M17[M17 Privacy] --> M06
    M17 --> M12
```

## 4. Request lifecycle — coach message (the most important path)

```mermaid
sequenceDiagram
    autonumber
    participant App as Flutter app
    participant API as CoachController
    participant Safe as SafetyService
    participant Ctx as ContextBuilder
    participant LLM as LlmGateway
    participant V as Vertex AI
    participant DB as PostgreSQL

    App->>App: decrypt last 7 days of entries locally
    App->>API: POST /coach/messages {text, conversationId, trackingContext}
    API->>API: auth, rate limit, daily cap check
    API->>Safe: classify(text)
    Safe->>LLM: classify (Flash-Lite)
    LLM->>V: generateContent
    V-->>LLM: {level, labels}
    alt level >= 2
        Safe-->>API: CRISIS
        API-->>App: SSE "safety" {level, hotlines[]} (scripted card)
        API->>LLM: crisis reply (strongest model, crisis prompt)
    else level 0-1
        API->>Ctx: build(userId, trackingContext)
        Ctx->>DB: profile, plan, memory summary, last 20 messages
        API->>LLM: chat stream (Flash)
    end
    LLM->>V: streamGenerateContent
    V-->>App: SSE "token" ... "done"
    API->>DB: save messages (contentEnc), token counts
    API->>API: enqueue memory.summarize (debounced 10 min)
```

## 5. Environments

| Env | API | DB | AI | Notes |
| --- | --- | --- | --- | --- |
| local | `localhost:3000` | Docker Postgres 16 + Redis | `LLM_PROVIDER=fake` (deterministic stub) or Vertex dev project | Seed script with hotlines + test users |
| staging | Cloud Run `turnwell-api-staging` | Cloud SQL small | Vertex (staging project) | TestFlight / Play internal builds |
| prod | Cloud Run `turnwell-api` (min 1 instance) | Cloud SQL HA later | Vertex (prod project) | |

## 6. Key cross-cutting services (backend `common/`)

| Service | Purpose |
| --- | --- |
| `CryptoService` | KMS envelope encryption: `encrypt(userId, plaintext)`, `decrypt(userId, ciphertext)`; DEK cache 5 min |
| `RedactingLogger` | pino with redaction paths |
| `RateLimitGuard` | Redis-backed throttler with per-route overrides |
| `ClockService` | User-timezone day math (`todayFor(userId)`) |
| `IdService` | UUIDv7 |
| `AuditService` | Writes `system.audit_logs` |
| `FeatureFlags` | Config-table flags (`community.enabled`, `coach.model`) |
