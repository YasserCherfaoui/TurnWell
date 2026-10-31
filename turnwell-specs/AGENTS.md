# AGENTS.md — Instructions for AI coding assistants

> Read this file first. It is the contract for anyone (human or AI) writing code for **TurnWell**.
> If a module spec contradicts this file, this file wins. If this file is silent, follow `02-conventions.md`.

## 1. What we are building

TurnWell is a private, AI-coached mobile app (18+, global) that helps people quit **bad habits** (secular mode)
or **sins** (Islamic-guided mode). Core parts: tracking, a proactive AI coach, an SOS flow for urges, and
anonymous moderated community posts. Full context: `00-product-overview.md`.

## 2. Stack (do not substitute without asking)

| Layer | Choice |
| --- | --- |
| Mobile | Flutter 3.x (Dart 3), iOS + Android. Web later. |
| State | Riverpod 2 with `riverpod_generator` (`@riverpod`) |
| Routing | `go_router` |
| Local DB | Drift + SQLCipher (`sqlcipher_flutter_libs`) |
| Secure storage | `flutter_secure_storage` |
| Crypto (client) | `cryptography` (XChaCha20-Poly1305, Argon2id, HKDF) |
| HTTP | `dio` + generated OpenAPI client |
| Backend | NestJS 10+ (TypeScript, strict), modular monolith |
| ORM | Prisma 5 |
| Database | PostgreSQL 16 (Cloud SQL) |
| Queue / cache | Redis (Memorystore) + BullMQ |
| AI | Gemini on **Vertex AI paid tier only**, via the internal `LlmProvider` abstraction |
| Cloud | GCP, region `europe-west1` |
| Push | FCM (Android) + APNs via FCM (iOS) |

## 3. Non-negotiable rules

### Privacy (the product IS privacy)
1. **Never log user content.** No message text, journal text, post text, prompts, or AI replies in logs, analytics,
   Sentry breadcrumbs, or error messages. Log IDs, counts, durations, status codes only.
2. **Journal + tracking payloads are end-to-end encrypted on the device** (`M06-encryption-sync.md`). The server stores
   ciphertext blobs. Never add a server feature that requires reading them.
3. **Two identities:** `User.id` (account) and `Pseudonym.id` (community). Community tables must NEVER contain `userId`.
   The only link is `community.identity_links`, encrypted with a separate KMS key, read only by `SafetyService` and
   `LegalEscalationService`.
4. **No PII in LLM prompts.** No email, name, phone, account ID. Use the coach name and the user's chosen context only.
5. **Never call the Gemini free tier / AI Studio key in staging or prod.** Vertex AI only.
6. **Lock-screen notification text is always the user's discreet template**, never AI-generated content.

### Safety
7. Every user-authored text that reaches the AI or the community passes `SafetyService.classify()` first (`M09`).
8. Crisis responses (levels 2–3) always include the **scripted** hotline card. Never rely on the model to show it.
9. The coach must never claim to be a therapist, doctor, or religious scholar, and never issue religious rulings.

### UX
10. **No shame UI.** Never use red for slips (use neutral grey). Never show "streak lost", "failed", or reset-to-zero
    as the main metric. Use `% clean days` and trends.
11. Every screen must work in **RTL (Arabic)** and with **large text** (Dynamic Type up to 200%).
12. All user-visible strings go through `AppLocalizations` (ARB). No hard-coded strings.

### Engineering
13. Feature-first folders. One module = one Flutter feature folder + one NestJS module.
14. Every endpoint: DTO validation (`class-validator`), auth guard, rate limit, OpenAPI decorators.
15. Every new table: Prisma migration, `createdAt`, `updatedAt`, UUIDv7 id.
16. Write tests listed in each module's **Tests** section before marking a feature done.
17. Release tags: only build features tagged **v1** unless told otherwise. v2/v3 items are documented so you can
    leave extension points, not so you build them now.

## 4. How the spec pack is organised

```
README.md                  ← index + reading order
AGENTS.md                  ← this file
00-product-overview.md     ← vision, modes, personas, feature list (F-IDs)
01-architecture.md         ← system diagram, module map, request lifecycle
02-conventions.md          ← code style, folders, API format, errors, testing
03-data-model.md           ← all tables, ER diagrams; schema/schema.prisma is the source of truth
04-design-system.md        ← liquid glass tokens, components, screen inventory
modules/M01 … M18          ← one file per module (same template)
prompts/                   ← AI system prompt blocks and classifier prompts
diagrams/                  ← rendered SVG of every Mermaid diagram (for humans)
```

## 5. Working method for AI assistants

1. Before coding a module, read: `AGENTS.md` → `02-conventions.md` → the module file → its **Dependencies**.
2. Implement in this order inside a module: Prisma models → NestJS DTOs/service/controller → tests →
   Flutter data layer → providers → screens → widget tests.
3. Use the exact endpoint paths, field names, enum values and error codes from the specs.
4. When something is ambiguous, check the **Open questions** section of the module; if not listed, choose the
   simplest option that respects section 3 and leave a `// SPEC-QUESTION:` comment.
5. Build order for v1 is in `README.md` (matches the [MVP build plan](../MVP.md)).
