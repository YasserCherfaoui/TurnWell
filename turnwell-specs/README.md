# TurnWell — Spec Pack for AI-assisted development

Private, AI-coached mobile app (Flutter + NestJS) that helps adults quit bad habits or sins.
This folder is written to be read by an AI IDE (Cursor, Windsurf, Claude Code, Copilot, etc.) **and** by humans.

## Start here

1. **[AGENTS.md](AGENTS.md)** — rules every AI assistant must follow (stack, privacy, safety, UX). Copy it to the repo root.
   - Cursor: also copy to `.cursor/rules/turnwell.mdc` · Claude Code: copy as `CLAUDE.md` · Copilot: `.github/copilot-instructions.md`
2. [00-product-overview.md](00-product-overview.md) — what and why, modes, feature IDs (F-xx)
3. [01-architecture.md](01-architecture.md) — system + module map + request lifecycle
4. [02-conventions.md](02-conventions.md) — folders, naming, API/error format, testing, module template
5. [03-data-model.md](03-data-model.md) + [schema/schema.prisma](schema/schema.prisma) — every table
6. [04-design-system.md](04-design-system.md) — liquid glass tokens, components, routes, wireframes
7. The module you are building (below) + the prompts it uses

## Modules

| ID | Module | Release | File |
| --- | --- | --- | --- |
| M01 | Auth & accounts | v1 | [modules/M01-auth-accounts.md](modules/M01-auth-accounts.md) |
| M02 | App lock & disguise | v1 | [modules/M02-app-lock-disguise.md](modules/M02-app-lock-disguise.md) |
| M03 | Onboarding & plan | v1 | [modules/M03-onboarding-plan.md](modules/M03-onboarding-plan.md) |
| M04 | Tracking (items, check-ins, slips, progress) | v1 | [modules/M04-tracking.md](modules/M04-tracking.md) |
| M05 | Journal | v1 | [modules/M05-journal.md](modules/M05-journal.md) |
| M06 | Encryption & sync | v1 | [modules/M06-encryption-sync.md](modules/M06-encryption-sync.md) |
| M07 | AI coach | v1 | [modules/M07-ai-coach.md](modules/M07-ai-coach.md) |
| M08 | LLM gateway | v1 | [modules/M08-llm-gateway.md](modules/M08-llm-gateway.md) |
| M09 | Safety & crisis | v1 | [modules/M09-safety-crisis.md](modules/M09-safety-crisis.md) |
| M10 | SOS (urge help) | v1 | [modules/M10-sos.md](modules/M10-sos.md) |
| M11 | Nudges & notifications | v1 | [modules/M11-nudges-notifications.md](modules/M11-nudges-notifications.md) |
| M12 | Community | v1 posts / v2 circles, DMs | [modules/M12-community.md](modules/M12-community.md) |
| M13 | Moderation & console | v1 basic | [modules/M13-moderation-console.md](modules/M13-moderation-console.md) |
| M14 | Settings & personalization | v1 | [modules/M14-settings-personalization.md](modules/M14-settings-personalization.md) |
| M15 | Subscriptions | v2 (v1: limits) | [modules/M15-subscriptions.md](modules/M15-subscriptions.md) |
| M16 | Localization | v1 | [modules/M16-localization.md](modules/M16-localization.md) |
| M17 | Privacy: export, delete, panic wipe | v1 | [modules/M17-privacy-export-delete.md](modules/M17-privacy-export-delete.md) |
| M18 | Infrastructure & DevOps | v1 | [modules/M18-infrastructure-devops.md](modules/M18-infrastructure-devops.md) |

AI prompts: [prompts/README.md](prompts/README.md). Rendered diagrams and screen mockups: [diagrams/](diagrams/).

## v1 build order (matches the [MVP build plan](../MVP.md), start Mon 2 Nov 2026)

| Week | Build | Modules |
| --- | --- | --- |
| 1 | Monorepo, GCP, Prisma, glass theme, auth | M18, M01, design system |
| 2 | Lock + disguise, i18n, mode + coach setup screens | M02, M16, M03 (screens) |
| 3–4 | LLM gateway, safety classifier, coach chat, onboarding conversation + plan | M08, M09 (classifier), M07, M03 |
| 5 | Encryption, local DB, sync, tracking, journal | M06, M04, M05 |
| 6 | SOS + crisis protocol + trusted contact | M10, M09 |
| 7 | Push, local reminders, nudge engine | M11 |
| 8–9 | Pseudonyms, posts, replies, moderation pipeline, basic console | M12, M13 |
| 10 | Export/delete/panic wipe, Night theme, settings, accessibility | M17, M14 |
| 11 | Closed beta, security review | all |
| 12 | Store submission | M18 |

## How to prompt your AI IDE (examples)

> "Read AGENTS.md, 02-conventions.md and modules/M04-tracking.md. Implement the NestJS part of M04 (sections 7–9)
> with Prisma models from schema/schema.prisma, and the e2e tests in section 14."

> "Read AGENTS.md, 04-design-system.md and modules/M10-sos.md. Build the Flutter SOS overlay (section 4 wireframes,
> section 10 folder tree) with widget tests in LTR and RTL."

## Conventions used in this pack

- **Release tags:** v1 = build now; v2/v3 = documented extension points only.
- **F-xx** = feature IDs; **M-xx** = modules; error codes `MODULE_REASON`.
- Diagrams are Mermaid inside the Markdown (the source); SVG renders are in `diagrams/`.
