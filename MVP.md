# TurnWell — MVP v1 Build Plan

Oct 30, 2026 · @ladj ben

## TL;DR

TurnWell v1 can ship in about 12 weeks solo (start Mon 2 Nov 2026, store submission week of 18 Jan 2027), if the scope stays exactly as listed below.

- **v1 core:** onboarding conversation + plan, tracking + journal, AI coach, proactive nudges, SOS + crisis protocol, reach-out posts with AI moderation, app lock + disguise, Dawn + Night themes, 4 languages.
- **Deferred to v2:** circles, DMs, accountability partners, subscriptions, voice, weekly reflection, Oasis theme.
- **Riskiest parts:** end-to-end encryption + sync (week 5), crisis safety quality (week 6), and App Store review of a mental-wellness app with user content (week 12). Each has a mitigation below.
- **Estimated running cost at launch:** roughly 125–280 USD/month for GCP + AI at up to 1,000 monthly active users.
- **Not blocked by code:** the legal entity must be decided by week 8 to open store accounts in time.
- Full product details live in [*TurnWell — Product Specs Book*](./SPECS.md); feature IDs (F-xx) below refer to it.

## Scope

v1 includes 36 features from the specs book; everything else waits until real users have tried the core loop.

| Module | In v1 | Deferred |
| --- | --- | --- |
| Account and onboarding | F-01 sign-up, F-02 app lock, F-03 language, F-04 mode, F-05 coach setup, F-06 onboarding chat, F-07 trusted contact, F-08 pseudonym | — |
| Tracking | F-10 items, F-11 check-in, F-12 slip log, F-13 progress, F-15 journal (text + on-device voice transcription) | F-14 growth art, F-16 replacements, F-17 positive practices, F-18 widget |
| AI coach | F-20 chat (15 msg/day free), F-21 nudges, F-22 plan, F-25 tone | F-23 weekly reflection, F-24 voice, F-26 lessons |
| SOS | F-30 button, F-31 urge flow, F-32 outcome, F-33 crisis path | — |
| Community | F-40 reach-out posts, F-41 supportive replies | F-42 circles, F-43 DMs, F-44 partners, F-45 milestones, F-46 mentors |
| Privacy | F-50 disguise, F-51 decoy PIN, F-52 discreet notifications, F-53 panic wipe, F-54 export/delete | F-55 dashboard |
| Settings | F-60 Dawn + Night themes, F-61 coach settings | Oasis theme, F-62 prayer times |
| Billing | F-70 free limits (no payments) | F-71 premium |
| Moderation | F-80 AI pre-moderation, F-81 report, F-82 basic console | Full console, appeals UI |
| Notifications | F-90 local reminders, F-91 push | — |

**Scope rule:** any new idea during the build goes to a v2 list, not into v1. The only exceptions are safety and privacy fixes.

## Timeline

The build runs 11 weeks from 2 November, then store submission on 18 January with up to two weeks for review.

&#91;embedded content: MVP v1 timeline · 9 build phases, 2 milestones\]

The community phase spans the year-end holidays on purpose: it doubles as the schedule buffer.

## Sprint tasks

Each week ends with a working build on a real phone; tick tasks off as you go.

### Week 1 (2 Nov) — Foundations

- [ ] Create monorepo, Flutter app, NestJS API, CI (section below)
- [ ] GCP project: Cloud Run, Cloud SQL (PostgreSQL 16), Memorystore, Secret Manager, KMS, europe-west1
- [ ] Prisma schema v1 (account, private, community schemas) + first migration
- [ ] Flutter core: Riverpod, go\_router, Dio, theme tokens, `GlassSurface` + `MeshBackground`
- [ ] Auth: email + Google + Apple, JWT + refresh, 18+ gate (F-01)

### Week 2 (9 Nov) — Lock, disguise, i18n

- [ ] PIN + biometric lock, auto-lock (F-02)
- [ ] Alternate icons + decoy PIN screen (F-50, F-51)
- [ ] ARB files for EN / AR / FR / ES, RTL check on every screen built so far (F-03)
- [ ] Mode choice + coach setup screens (F-04, F-05)
- [ ] Bottom nav shell: Home, Coach, Progress, Community, Me

### Weeks 3–4 (16–29 Nov) — AI coach and onboarding

- [ ] `llm` module: `LlmProvider` interface, Gemini (Vertex AI) implementation, model routing config, token/cost logging
- [ ] Prompt assembly: core, persona, mode, methods, user context, guardrails (versioned files)
- [ ] `safety` module: classifier on every message (levels 0–3), hotline table seeded for 30 countries + fallback
- [ ] Coach chat with streaming replies, daily message limit (F-20, F-25)
- [ ] Onboarding conversation → validated JSON → editable plan screen (F-06, F-22)
- [ ] Memory: rolling summary job, 90-day message cleanup
- [ ] Test set: 50 scripted conversations (normal, sins mode, crisis, jailbreak attempts) run before every prompt change

### Week 5 (30 Nov) — Tracking, journal, encryption

- [ ] Device master key + Keychain/Keystore storage; recovery passphrase with Argon2id wrap
- [ ] Drift + SQLCipher local DB; encrypted payload format
- [ ] Sync API (`push` / `pull?since=`) with versions + last-write-wins
- [ ] Items, check-in, slip log, progress screen (F-10–F-13)
- [ ] Journal with on-device voice transcription (F-15)

### Week 6 (7 Dec) — SOS and crisis

- [ ] SOS button + urge flow: breathe, ride the urge timer with coach, distract, dhikr, outcome (F-30–F-32)
- [ ] Crisis card UI, hotline lookup by country, trusted-contact setup and one-tap message (F-07, F-33)
- [ ] Run the crisis test set in all 4 languages; fix until every level-2/3 case shows the hotline

### Week 7 (14 Dec) — Nudges and notifications

- [ ] FCM/APNs setup, device registration, discreet templates (F-52, F-91)
- [ ] Local check-in reminders (F-90)
- [ ] Nudge engine: Cloud Scheduler every 15 min, limits (3/day default, quiet hours, 2-hour gap, back-off) (F-21)
- [ ] On-device risky-window detection → synced nudge schedule

### Weeks 8–9 (21 Dec – 3 Jan) — Community and moderation

- [ ] Pseudonym generation + identity\_links with separate KMS key (F-08)
- [ ] Reach-out posts: visibility, content warning, feed ("People like me" uses category match) (F-40)
- [ ] Replies + reactions, AI screening before display (F-41)
- [ ] Moderation pipeline: rules + AI labels + decisions; report flow (F-80, F-81)
- [ ] Basic Moderator console (Next.js or a simple NestJS-served admin; queue, hide/restore, mute, audit) (F-82)
- [ ] Holiday weeks: plan reduced pace; this is the schedule buffer

### Week 10 (4 Jan) — Privacy and polish

- [ ] Panic wipe, export (JSON + PDF), delete account (F-53, F-54)
- [ ] Night Sanctuary theme + auto day/night switch (F-60, F-61)
- [ ] Accessibility pass: contrast on glass, Dynamic Type, screen reader labels
- [ ] Performance pass on low-end Android (blur fallbacks)

### Week 11 (11 Jan) — Closed beta

- [ ] TestFlight + Play internal testing with 20–50 testers (mix of languages and both modes)
- [ ] In-app feedback button; fix critical bugs daily
- [ ] Security review: logs redacted, no content in analytics, rate limits, pen-test checklist (OWASP MASVS basics)

### Week 12 (18 Jan) — Store submission

- [ ] Store listings in 4 languages, screenshots, privacy labels (App Privacy / Data safety)
- [ ] Review notes for Apple: moderation, reporting, blocking, crisis resources, 17+ rating
- [ ] Submit; keep 1–2 weeks for review rounds

## Repo and project setup

One monorepo keeps the app, API, admin console and shared contracts in sync for a solo developer.

```text
turnwell/
├── apps/
│   ├── mobile/            # Flutter (iOS + Android)
│   │   └── lib/
│   │       ├── core/      # theme, glass widgets, crypto, db, network, l10n
│   │       └── features/  # onboarding, tracking, coach, sos, community, settings, lock
│   ├── api/               # NestJS modular monolith
│   │   └── src/modules/   # auth, users, sync, coach, llm, safety, nudges, community, moderation, admin
│   └── admin/             # Moderator console (web)
├── packages/
│   ├── prompts/           # versioned system prompt blocks + test conversations
│   └── contracts/         # OpenAPI spec → generated Dart + TS clients
├── infra/                 # Terraform for GCP
└── .github/workflows/     # CI: lint, test, build, deploy to Cloud Run
```

| Item | Choice |
| --- | --- |
| Environments | `dev` (local Docker: Postgres + Redis), `staging` and `prod` on GCP |
| API client | OpenAPI generated from NestJS → Dart client via openapi-generator |
| Testing | API: Jest unit + e2e on Postgres testcontainer; app: widget tests for core flows; prompt test set run in CI |
| App builds | Codemagic (iOS without a Mac in CI) or Fastlane |
| Error tracking | Sentry (PII scrubbing on, no request bodies) |
| Feature flags | Simple config table or Firebase Remote Config (no user data) |

## Running costs estimate

At up to 1,000 monthly active users, TurnWell should cost roughly 125–280 USD per month, with AI as the item that grows with usage. Figures are approximate list prices; validate them in the GCP pricing calculator.

| Item | Assumption | Approx. USD / month |
| --- | --- | --- |
| Cloud Run (API) | 1 min instance, low traffic | 15–40 |
| Cloud SQL PostgreSQL | Small instance + backups | 30–60 |
| Redis | Memorystore basic 1 GB (or a cheaper managed Redis) | 10–40 |
| KMS, Secret Manager, Storage, Scheduler | Low volume | 5–15 |
| Gemini coach (Flash) | \~60 messages per user per month, \~3k input + 300 output tokens each ([Flash: $0.30 / $2.50 per 1M tokens](https://www.morphllm.com/gemini-api-pricing)) | 60–110 |
| Gemini safety + moderation (Flash-Lite) | Every message and post classified | 5–15 |
| Push (FCM/APNs) | — | 0 |
| **Total** |  | **\~125–280** |

One-off: Apple Developer Program 99 USD/year, Google Play 25 USD once.

**Cost controls:** context caching for the fixed system prompt, keep the memory summary short (about 1,500 tokens), free-tier daily message cap, batch API for summaries, and a per-user daily token ceiling with alerts.

## Risks and mitigations

The three risks most likely to delay launch are encryption complexity, AI safety quality and App Store review.

| Risk | Impact | Mitigation |
| --- | --- | --- |
| E2E encryption + sync takes longer than 1 week | Delays tracking and journal | Use proven packages only (`cryptography`, SQLCipher); last-write-wins sync, no merge logic in v1; if needed, ship v1 with KMS encryption at rest for tracking and add E2E in v1.1 (journal stays E2E) |
| AI says something harmful in a crisis | User harm, store removal | Classifier on every message, scripted hotline messages that never depend on the model, strongest model for crisis, 50-conversation test set in CI, crisis cases reviewed in 4 languages |
| App Store rejection (user content, mental health) | 1–3 week delay | Report + block + moderation from day one, 17+ rating, clear disclaimer, review notes explaining moderation and crisis flow, submit a TestFlight build early to catch issues |
| Legal entity not ready | Cannot publish or get paid | Decide by week 8; store accounts can take 1–2 weeks to verify |
| Solo burnout / scope creep | Missed date | Scope rule, holiday weeks as buffer, weekly demo on a real phone |
| AI costs higher than expected | Margin | Daily caps, caching, per-user token ceiling, alerts at 80% of monthly budget |
| Liquid glass slow on low-end Android | Bad reviews | Max 2 blur layers, solid-glass fallback by device class |
| Bad actors in community | Trust loss | AI screening before display, no DMs in v1, zero-tolerance bans, Moderator SLA |

## Launch checklist

v1 is ready to submit when every item below is ticked.

**Product**

- [ ] A new user reaches a personal plan in under 10 minutes, in all 4 languages
- [ ] Check-in, slip, SOS and journal work fully offline and sync after reconnecting
- [ ] Nudges respect max/day, quiet hours and discreet lock-screen text
- [ ] Disguise icon, decoy PIN and panic wipe tested on iOS and Android

**Safety**

- [ ] Every level-2/3 test conversation shows the correct country hotline
- [ ] Trusted-contact message sends only on the user's tap
- [ ] Posts with crisis signals are hidden from others and the author gets the crisis card
- [ ] Report, block, and Moderator actions work end to end; audit log records them

**Privacy and security**

- [ ] No message content in logs, analytics or Sentry
- [ ] Journal entries unreadable in the database (verified manually)
- [ ] Restore on a new phone works with the recovery passphrase
- [ ] Export and account deletion work
- [ ] Vertex AI paid tier, EU region, data-processing terms accepted

**Store and legal**

- [ ] Legal entity + developer accounts verified
- [ ] Privacy policy, terms, "not a medical service" disclaimer published
- [ ] App Privacy (iOS) and Data safety (Android) forms match actual data use
- [ ] 17+ / Mature rating, review notes, listings and screenshots in 4 languages
- [ ] TurnWell trademark search done
