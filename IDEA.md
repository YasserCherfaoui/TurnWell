# TurnWell — Ideation Decisions Log
_Last updated: 2026-10-30_

**Specs:** [TurnWell — Product Specs Book](https://claude.ai/artifact/HzQKcqhpUaRZMPrY4SGQoV) · [TurnWell — MVP v1 Build Plan](https://claude.ai/artifact/NhS8XLiS3HfVzasqdcVQ5C)

## Product
- **Name:** TurnWell (formal trademark / store-name check still pending)
- **Concept:** Track bad habits and sins, with a proactive AI coach (therapeutic techniques, not presented as a licensed therapist), moderated anonymous communities, and public "reach out" feeling posts.
- **Modes:** Users pick at onboarding what affects them more — **Sins** (Islamic-guided) or **Bad habits** (secular). Users can track both.
- **Faith approach:** Islamic practices only (dhikr, salah, tawbah, du'a...). No religion selector, no fatwas. The AI sticks to well-established practices and never issues rulings.
- **Onboarding:** A short conversation with the AI, where the user describes what they want to fix → a personalized plan is generated. The AI maps any habit onto an internal taxonomy.
- **AI coach persona:** the user picks a male or female persona and voice, and gives the coach a name.
- **AI:** Proactive. Nudges: 3/day by default, adjustable, plus quiet hours. Voice conversations wanted (premium, v2).
- **Audience:** 18+ only. Global.
- **UI languages v1:** English, Arabic (RTL), French, Spanish. English is the fallback. The AI converses in any language.

## Community
- Same-gender circles: **yes** (option)
- DMs: **yes**, with mutual opt-in, no images or links at first, AI scanning for grooming, scams and solicitation, and one-tap block/report
- Post visibility set by the user: **Public** or **People like me**
- Moderation: AI-first; a dedicated **Moderator** role (human) can step in on reports

## Safety
- Crisis protocol: local hotlines by country + safe script + a **trusted contact** the user sets up in advance

## Privacy
- Level **(a)**: the journal and tracking are end-to-end encrypted. AI chats are encrypted at rest, processed by the LLM, never read by humans.
- Sign-up: email, Google, Apple. The community pseudonym is never linked to the private identity.
- Lock-screen notification text: user-customizable
- Offline-first tracking and journal
- App disguise (alternate icon, PIN, decoy screen): **in v1**

## Business
- Freemium at launch, subscriptions later. Free: tracking, SOS, community, limited AI messages per day. Premium: unlimited AI, deep insights, voice.
- Publishing legal entity: **TBD** (needed by week 8 of the build)

## Design
- Liquid glass, three themes the user can choose: **Dawn**, **Night Sanctuary**, **Oasis**
- Defaults: Dawn by day, Night Sanctuary by night

## Tech
- Flutter (iOS + Android at launch, web later)
- NestJS + PostgreSQL on GCP
- LLM: Gemini (**paid tier / Vertex AI only** — the free tier may use data for training), behind a provider abstraction; model tiers by task

## Builder constraints
- Solo developer, ASAP → two docs: a full specs book + an MVP v1 build plan (~12 weeks)