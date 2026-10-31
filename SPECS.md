# TurnWell — Product Specs Book

Oct 30, 2026 · @ladj ben

## TL;DR

TurnWell is a private, AI-coached app (18+, global) that helps people quit bad habits or sins. It has four parts: tracking, a proactive AI coach, an SOS button for urges, and anonymous same-habit communities.

- **Two modes:** Sins (Islamic-guided practices) and Bad Habits (secular). Users can track both.
- **AI coach:** a short onboarding conversation produces a personal plan. The coach then checks in proactively (3 nudges a day by default, quiet hours). The user chooses its name, gender and voice.
- **Privacy is the product:** the journal and tracking are end-to-end encrypted; the community pseudonym is never linked to the real account; app disguise and a PIN are in v1.
- **Safety:** crisis detection, local hotlines, a trusted contact, AI-first moderation with a human Moderator role.
- **Stack:** Flutter (iOS + Android), NestJS + PostgreSQL on GCP, Gemini on the paid tier behind a provider abstraction.
- **Look:** liquid glass with three selectable themes. Dawn is the default by day, Night Sanctuary by night.
- **Business:** freemium at launch (limited AI messages per day), subscriptions later.
- The MVP v1 scope and build order are in the companion doc, [*TurnWell — MVP v1 Build Plan*](./MVP.md).

## 1. Vision and principles

TurnWell is the one place a person can be fully honest about what they struggle with, and get daily, personal help to change it.

**Positioning:** a judgment-free recovery companion for habits and sins, combining a habit tracker, an AI coach and peer support, built on stronger privacy than any social or wellness app.

**Product principles** (every feature decision is checked against these):

1. **Trust first.** Collect the minimum. Encrypt by default. Never show a real identity in the community. No ads, no data selling, ever.
2. **No shame.** A slip is data, not failure. No red, no streaks resetting to zero, no guilt language. Progress shows as a percentage and a trend.
3. **Prevent, don't punish.** The AI steps in *before* risky moments (learned triggers and times), not only after a slip.
4. **Human connection over dependency.** The coach pushes toward real people: community, an accountability partner, a trusted contact. It never presents itself as the user's only support.
5. **Safe by design.** Crisis detection, strict DM rules, 18+ only, AI-first moderation with human escalation.
6. **Not a therapist.** The AI uses evidence-based techniques (CBT, Motivational Interviewing, ACT, urge surfing). It is never presented as a licensed professional and never diagnoses.
7. **Faith-respectful, not preachy.** Sins mode suggests well-established Islamic practices and never issues religious rulings (fatwas).

## 2. Users and modes

TurnWell serves adults (18+) worldwide who want to stop a behavior they hide from people around them.

### Personas

| Persona | Profile | Main struggle | What they need most |
| --- | --- | --- | --- |
| Yacine, 24 | Muslim student, Algiers | Pornography, missed prayers | Total secrecy, faith-based guidance, same-gender circle |
| Laura, 31 | Secular office worker, Madrid | Smoking, doomscrolling | Simple tracking, urge help at work, light coaching |
| Omar, 38 | Father, Paris | Anger, gambling | Night-time support, a trigger plan, an accountability partner |
| Amina, 27 | Muslim nurse, London | Backbiting, emotional eating | A judgment-free place to talk, people like her |

### Modes

At onboarding, the user chooses what affects them more. Both modes can run at the same time; each tracked item belongs to one.

|  | Sins mode | Bad Habits mode |
| --- | --- | --- |
| Framing | Sin, tawbah (repentance), getting closer to Allah | Habit, behavior change, wellbeing |
| Coach suggestions | Dhikr, salah, Qur'an, du'a, fasting, plus psychology techniques | Psychology techniques only |
| Vocabulary | "Sin", "slip", "return" | "Habit", "slip", "reset" |
| Extras | Prayer-time aware nudges, Islamic reminders | Neutral reminders |
| Community | Faith circles (same-gender option) | General circles (same-gender option) |

**Rules for Sins mode:** it is presented as Islamic-guided. A non-Muslim who picks it sees a one-line notice and can switch at any time. The AI suggests only well-established practices and says "ask a scholar" for anything that needs a ruling.

### Habit and sin taxonomy

Users never pick from a giant menu. The onboarding AI maps free text onto an internal taxonomy. A user can also add a custom item.

| Category | Examples |
| --- | --- |
| Sexual | Pornography, compulsive masturbation, illicit relationships |
| Substances | Smoking, vaping, alcohol, drugs |
| Digital | Doomscrolling, gaming, social media |
| Speech | Lying, backbiting (ghiba), cursing, gossip |
| Emotional | Anger, envy, arrogance |
| Financial | Gambling, impulse buying, riba (interest) |
| Worship (Sins mode) | Missed prayers, delaying prayer, neglecting Qur'an |
| Body | Emotional eating, poor sleep, sedentary life |
| Productivity | Procrastination |
| Custom | Anything the user names |

## 3. Feature specs

The product is grouped into 10 modules. Each feature has an ID and a release tag: **v1** (MVP), **v2** or **v3**.

### 3.1 Account and onboarding

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-01 | Sign-up | Email + password, Google, Apple. Age confirmation (18+) required. Email is never shown to anyone. | v1 |
| F-02 | App lock | PIN and/or biometric, required after sign-up. Auto-lock after 1 minute in background (adjustable). | v1 |
| F-03 | Language | Auto-detected. UI in English, Arabic (RTL), French, Spanish; fallback English. | v1 |
| F-04 | Mode choice | "What affects you more: sins or bad habits?" Can enable both. | v1 |
| F-05 | Coach setup | User picks coach gender (male/female), voice, and a name (default suggestions per language, e.g. Noor, Sam). | v1 |
| F-06 | Onboarding conversation | 5–10 minute chat with the coach: what to fix, when it happens, triggers, past attempts, motivation. Output: structured profile + a personal plan the user can edit. | v1 |
| F-07 | Trusted contact | Optional: name + phone/email of a person to alert in a crisis. Can be skipped and added later. | v1 |
| F-08 | Community pseudonym | Auto-generated anonymous name + abstract avatar. Created on first community use, never linked publicly to the account. | v1 |

### 3.2 Tracking

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-10 | Tracked items | Each item: name, category, mode, goal (quit / reduce to X per week), why it matters to the user. | v1 |
| F-11 | Daily check-in | One screen per item: clean / slipped / partly. Optional mood (5 levels) and a note. Takes under 20 seconds. | v1 |
| F-12 | Slip log | When a slip is logged: time (auto), where, mood before, trigger (chips + free text), who was around. All optional. | v1 |
| F-13 | Progress view | % clean days (7/30/90 days), trend line, best run, total clean days. No reset-to-zero counter as the main number. | v1 |
| F-14 | Visual growth | Theme-specific progress art (rising sun, stars, garden) that grows with clean days and never "dies" after a slip. | v2 |
| F-15 | Journal | Free text + voice notes (transcribed on device). End-to-end encrypted. | v1 |
| F-16 | Replacement habits | For each item, 1–3 healthy substitutes suggested by the coach and tracked as positives. | v2 |
| F-17 | Positive practices (Sins mode) | Track salah on time, dhikr, Qur'an pages, fasting, as positives alongside sins. | v2 |
| F-18 | Home-screen widget | Today's check-in status + SOS button. | v2 |

### 3.3 AI coach

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-20 | Coach chat | Text chat with the personalized coach. Remembers the user's journey (summarized long-term memory). Free: 15 messages/day. Premium: unlimited. | v1 |
| F-21 | Proactive nudges | Coach starts the conversation: check-in reminders, pre-trigger nudges, encouragement after a hard day. Default max 3/day, adjustable 0–6, quiet hours (default 23:00–07:00). | v1 |
| F-22 | Personal plan | Generated at onboarding, revised weekly: daily routine, trigger-avoidance rules, replacement actions, faith practices if Sins mode. | v1 |
| F-23 | Weekly reflection | Sunday report: progress, patterns found (e.g. "80% of slips after 23:00"), one focus for next week. | v2 |
| F-24 | Voice conversation | Talk to the coach out loud with the chosen voice. Premium only. | v2 |
| F-25 | Tone setting | Gentle / balanced / firm. | v1 |
| F-26 | Learning modules | Short lessons (2–3 min): how habits work, urge surfing, tawbah, sleep. | v3 |

### 3.4 SOS (urge help)

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-30 | SOS button | Always visible on home. One tap opens the urge flow. | v1 |
| F-31 | Urge flow | Pick: breathe (guided 2 min), ride the urge (10 min timer + coach talking you through), distract (quick actions), dhikr (Sins mode), or message accountability partner. | v1 |
| F-32 | Urge outcome | After the flow: "Did the urge pass?" Logged as a win if yes. Wins count in progress. | v1 |
| F-33 | Crisis path | If the user's words show suicide or self-harm risk, switch to the crisis protocol (section 6). | v1 |

### 3.5 Community

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-40 | Reach-out posts | Post feelings anonymously. Visibility: Public (all users) or People like me (same habit categories). Optional content warning. | v1 |
| F-41 | Supportive replies | Replies are screened by AI before they appear. Reactions: support, same here, praying for you (Sins mode). No downvotes, no follower counts. | v1 |
| F-42 | Circles | Groups per habit category × language × mode. Same-gender option. Max \~500 members per circle, then split. | v2 |
| F-43 | Direct messages | Mutual opt-in only. Text only (no images, links, phone numbers). AI scans for grooming, scams, solicitation, sharing contact details. Block + report in one tap. | v2 |
| F-44 | Accountability partner | 1:1 anonymous pairing. Each user chooses what the partner sees (streak, check-in status). Partner gets SOS alerts if allowed. | v2 |
| F-45 | Milestones | Share a milestone (7, 30, 90 days) to a circle; others cheer. | v2 |
| F-46 | Verified mentors | Members with long recovery or professionals, with a badge and extra tools. | v3 |

### 3.6 Privacy and disguise

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-50 | App disguise | Alternate icon + name (e.g. "Daily Notes", "Calculator"). | v1 |
| F-51 | Decoy PIN | A second PIN opens a harmless fake screen (e.g. a to-do list). | v1 |
| F-52 | Discreet notifications | Lock-screen text is user-customizable; default "You have a new reminder". No sensitive words ever on the lock screen. | v1 |
| F-53 | Panic wipe | A gesture or PIN deletes all local data instantly; server data is kept encrypted for recovery unless the user chooses "delete account". | v1 |
| F-54 | Export and delete | Export all data (JSON + PDF). Delete account: permanent removal within 30 days. | v1 |
| F-55 | Privacy dashboard | Plain-language view of what is stored, where, and who can see it. | v2 |

### 3.7 Settings and personalization

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-60 | Themes | Dawn, Night Sanctuary, Oasis. Auto mode: Dawn by day, Night Sanctuary by night. | v1 (Dawn + Night), v2 (all) |
| F-61 | Coach settings | Rename, change gender/voice, tone, nudge count, quiet hours. | v1 |
| F-62 | Prayer times | Location or city-based, used for Sins-mode nudges. | v2 |

### 3.8 Subscriptions

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-70 | Freemium limits | Free: tracking, SOS, community, journal, 15 AI messages/day, 1 proactive nudge/day minimum guaranteed. | v1 |
| F-71 | Premium | Unlimited AI, voice, weekly reflection, deep insights, all themes. Monthly + yearly, via App Store / Google Play billing. | v2 |

### 3.9 Moderation tools

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-80 | AI pre-moderation | Every post, reply and DM is classified before delivery (section 7). | v1 |
| F-81 | Report | Any content or user, 1 tap, with reason. | v1 |
| F-82 | Moderator console | Web admin panel for the Moderator role: queue, context, actions, audit log. | v1 (basic) |

### 3.10 Notifications

| ID | Feature | Spec | Release |
| --- | --- | --- | --- |
| F-90 | Local reminders | Check-in reminders scheduled on device (work offline). | v1 |
| F-91 | Push | Coach nudges, replies, DMs, partner alerts via FCM/APNs, with discreet text. | v1 |

## 4. Key user flows

The core loop runs once a day, and the SOS path can start at any moment; both feed the same progress and plan.

&#91;embedded content: Core loop · daily check-in and SOS paths\]

A slip never ends the loop: it becomes data the coach uses to adjust the plan, while a resisted urge is counted as progress.

### 4.1 First launch (target: under 10 minutes)

1. Splash → language auto-detected → privacy promise screen (3 short points).
2. Sign up (email / Google / Apple) + 18+ confirmation.
3. Set PIN / biometric; optional disguise icon.
4. Choose mode: sins, bad habits, or both.
5. Set up the coach: gender, voice, name.
6. Onboarding conversation with the coach.
7. Review and edit the generated plan; choose check-in time and nudge settings.
8. Optional: trusted contact, recovery passphrase (strongly suggested).
9. Land on Home with the first check-in scheduled.

### 4.2 Reach-out post

1. Community tab → "Share how you feel".
2. Write the post; choose visibility (Public / People like me) and an optional content warning.
3. AI moderation (under 2 seconds): published, blurred, held for review, or crisis card shown privately.
4. Others reply; each reply is screened before it appears.
5. Author gets a discreet notification for new replies.

### 4.3 Direct message (v2)

1. User taps "Message" on a post or reply.
2. The other person gets a request and must accept.
3. Text-only chat; contact details, links and images are blocked automatically.
4. Either side can block or report at any time; a report freezes the thread for review.

## 5. AI system design

The coach is one persona backed by several model tiers, a safety layer that runs on every message, and a memory that summarizes rather than stores everything.

### 5.1 Model routing

| Task | Model tier | Why |
| --- | --- | --- |
| Safety classification (crisis, abuse) on every user message | Gemini Flash-Lite | Fast and cheap; runs before the coach replies |
| Community moderation (posts, replies, DMs, reports) | Gemini Flash-Lite | High volume, short texts |
| Coach chat, onboarding conversation, plan generation | Gemini Flash | Good quality per cost |
| Crisis conversations | Strongest available Gemini model + fixed scripted messages | Highest stakes; scripts guarantee hotlines are always shown |
| Weekly reflection, memory summaries | Gemini Flash (batch API, about 50% cheaper) | Not time-sensitive |
| Voice | On-device speech-to-text → text model → cloud text-to-speech (v2); real-time live audio API later | Cost and privacy |

**Rules:**

- **Paid tier only** (Vertex AI on GCP, same region as the database). The free tier may use prompts for training, which would break the privacy promise.
- **Provider abstraction:** NestJS `LlmProvider` interface (`chat`, `classify`, `summarize`, `embed`). Model names live in config, so changing provider is a config change.
- **Data minimization:** no name, email or account ID in prompts. The user is referenced by an internal session token only.
- **Retention:** prompts are not logged by the provider beyond abuse-monitoring defaults; see the open item in section 11 on zero data retention.

### 5.2 Persona and system prompt

The system prompt is assembled per request from versioned blocks:

1. **Core identity:** a warm, honest recovery coach; not a therapist; never diagnoses; encourages real-world support.
2. **Persona:** the user's chosen name, gender (affects Arabic and French grammar) and tone (gentle / balanced / firm).
3. **Mode block:** Sins mode adds Islamic framing and approved practices; Bad Habits mode stays secular.
4. **Methods:** CBT (reframing thoughts), Motivational Interviewing (open questions, reflecting back), ACT (accepting urges without acting), urge surfing, implementation intentions ("if X then Y").
5. **User context:** profile summary, active plan, last 7 days of check-ins, recent slips and triggers, long-term memory summary.
6. **Guardrails:** no medical or medication advice, no religious rulings, no sexual content, no judging language, always route crisis to the protocol.
7. **Language:** reply in the user's language; fall back to English.

### 5.3 Memory

| Layer | Content | Storage |
| --- | --- | --- |
| Short-term | Last \~20 messages of the current conversation | Encrypted in PostgreSQL |
| Profile | Structured facts from onboarding (items, triggers, motivations, schedule) | Encrypted JSON |
| Long-term summary | Rolling summary updated after each conversation (max \~1,500 tokens) | Encrypted text |
| Events | Check-ins, slips, urges, moods | Structured tables (used for patterns) |

Old raw messages are deleted after 90 days by default; only the summaries are kept. The user can view and delete what the coach remembers.

### 5.4 Proactive engine

A NestJS scheduler (Cloud Tasks / cron) decides every 15 minutes which users should get a nudge.

| Nudge type | Trigger | Example |
| --- | --- | --- |
| Check-in | User's chosen time, no check-in yet today | "How did today go?" |
| Pre-trigger | 15–30 min before a learned risky time (from slip history) | "Evenings have been hard lately. Want to plan tonight?" |
| Follow-up | Day after a logged slip | "Yesterday was tough. One small step today?" |
| Encouragement | Milestone reached or a good week | "7 clean days. That's real." |
| Faith (Sins mode) | Near prayer times, Friday, Ramadan | "Maghrib is in 20 minutes." |

Limits: max nudges/day (default 3, user range 0–6), quiet hours, no two nudges within 2 hours, nudge stops if the user ignores 5 in a row (then 1/day until they return). Nudge text is generated by AI but the lock-screen text is always the user's discreet template.

### 5.5 Onboarding conversation output

The onboarding chat ends with a structured JSON (validated by the backend), shown to the user as an editable plan:

```json
{
  "modes": ["sins", "habits"],
  "items": [
    {"category": "sexual", "label": "Pornography", "mode": "sins", "goal": "quit",
     "risky_times": ["23:00-01:00"], "triggers": ["alone in bed", "stress"],
     "motivation": "Feel clean before my marriage"}
  ],
  "plan": {
    "daily": ["Phone out of bedroom at 22:30", "Isha + 5 min dhikr"],
    "if_then": ["If urge at night, open SOS and leave the room"],
    "check_in_time": "21:30"
  }
}
```

## 6. Privacy, security and safety

The privacy promise to users is: **"Your journal and tracking are encrypted so only you can read them. Your coach conversations are processed by AI and never read by humans. The community never knows who you are."**

### 6.1 Data classes

| Class | Examples | Protection | Who can read |
| --- | --- | --- | --- |
| A. Private journal | Journal entries, voice notes, slip notes | End-to-end encrypted on device (key never leaves the user's devices in readable form) | User only |
| B. Tracking | Check-ins, slips, moods, items | End-to-end encrypted payloads; the server keeps only minimal metadata (dates, counts) needed for nudges | User only (server sees counts and times, not content) |
| C. AI conversations | Coach chats, memory summaries, profile | Encrypted at rest (Cloud KMS, per-user data key); decrypted only in memory to call the AI | The system, never humans; no admin UI shows them |
| D. Community | Posts, replies, DMs, pseudonym | Encrypted at rest; tied to a pseudonym ID, not the account ID | Members per visibility; Moderators for reported items |
| E. Account | Email, auth provider, subscription | Encrypted at rest, access-logged | System; admin only for support with audit log |

### 6.2 Key design decisions

- **Two identities:** `account_id` (real account) and `pseudonym_id` (community). The mapping lives in a separate table encrypted with a separate KMS key; only the crisis and legal-escalation services can read it, and every read is audited.
- **End-to-end keys:** a per-user master key is generated on the device and stored in the Keychain / Android Keystore. A backup copy is wrapped with a key derived from the user's recovery passphrase (Argon2id), so encrypted data can be restored on a new phone. Losing both the device and the passphrase = losing class A and B data (stated clearly at setup).
- **Offline-first:** class A and B data live in an encrypted local database (Drift + SQLCipher) and sync encrypted blobs to the server.
- **Transport:** TLS 1.3, certificate pinning in the app.
- **No third-party trackers:** no ad SDKs. Analytics are self-hosted or privacy-first, with event names only (never content).
- **Logs:** no message content in logs; request bodies redacted.
- **Deletion:** account delete wipes all classes within 30 days, including backups' rotation window.

**How the AI uses encrypted tracking data:** the app decrypts the relevant check-ins and slips on the device and attaches them to each coach request. The server forwards them to the AI without storing them. Risky-time patterns for nudges are computed on the device and synced only as a nudge schedule (times, no content).

### 6.3 Crisis protocol

Every user message (coach chat, journal-to-AI, community post) passes a safety classifier with four levels.

| Level | Signal | Action |
| --- | --- | --- |
| 0 — None | Normal | Continue |
| 1 — Distress | Hopelessness, intense shame | Coach switches to supportive mode, suggests talking to someone, offers trusted contact |
| 2 — Risk | Mentions of self-harm or suicidal thoughts | Fixed safe-message script + local hotline for the user's country + one-tap "Message my trusted contact" + calm coach conversation (strongest model) |
| 3 — Imminent | Plan, means, or timing stated | Same as 2, with emergency number shown first and a full-screen card; repeated check-in an hour later |

Rules: the coach never promises confidentiality from emergency services and never handles level 2–3 alone. The hotline directory is a maintained table per country, with a global fallback. The trusted contact message is pre-written by the user at setup and is never sent without the user's tap. In community posts, a level 2–3 post is hidden from the public and the author gets the crisis card privately.

### 6.4 Age and legal safety

- 18+ confirmation at sign-up; App Store / Play age rating 17+ / Mature.
- Signals of a minor in conversation (stated age, school grade) → account flagged for review and community access paused.
- Disclaimer at onboarding and in settings: "TurnWell is not a medical service or a substitute for professional help."

## 7. Moderation

AI checks every community message before it is delivered; humans with the Moderator role handle what the AI is unsure about and every appeal.

### 7.1 Pipeline

1. User submits a post, reply or DM.
2. Fast rules: blocked patterns (phone numbers, emails, URLs, social handles, payment requests) in DMs and replies.
3. AI classifier (Flash-Lite) returns labels + confidence: `safe`, `crisis`, `sexual`, `grooming`, `harassment`, `hate`, `scam`, `contact_sharing`, `graphic_relapse`, `spam`, `religious_attack`.
4. Decision:
   - High-confidence safe → published.
   - `graphic_relapse` → published blurred with a content warning.
   - `crisis` → hidden from others; author gets the crisis card (section 6.3).
   - High-confidence violation → blocked; author sees a kind explanation.
   - Uncertain → held, sent to the Moderator queue.
5. User reports go to the AI first (priority score), then to the queue. Grooming and minor-safety reports always reach a human.

### 7.2 Roles

| Role | Can do | Cannot do |
| --- | --- | --- |
| Member | Post, reply, react, report, block, DM (opt-in) | See anyone's real identity |
| Moderator | Review queue, see reported content with thread context, hide/restore content, warn, mute (24h/7d), suspend pseudonym, escalate | See email, account data, journal, coach chats; un-mask identities |
| Admin | Everything a Moderator can, plus manage Moderators, ban accounts, configure rules | Read classes A–C (no UI exists for it) |
| Legal escalation (Admin + 2nd approver) | Un-mask a pseudonym for a valid legal request or imminent-harm case | Act alone; every un-mask is audit-logged |

### 7.3 Enforcement ladder

| Strike | Action |
| --- | --- |
| 1 | Warning + content removed |
| 2 | Mute 24 hours |
| 3 | Mute 7 days, DMs disabled |
| 4 | Community ban (tracking and coach stay available) |
| Zero tolerance | Grooming, sexual solicitation, minors, doxxing → immediate account ban |

Users can appeal any action once; appeals always go to a human. Moderator SLA target: under 24 hours for normal queue, under 1 hour for grooming / safety.

## 8. Technical architecture

TurnWell runs as one Flutter app talking to one NestJS modular monolith on Cloud Run. This is the fastest setup for a solo builder, and modules can be split into services later.

&#91;embedded content: TurnWell architecture · 2 clients, 1 API, 7 managed services\]

The app holds private data encrypted on the device; the API only stores encrypted blobs, community data and the metadata needed for nudges.

### 8.1 Flutter app

| Concern | Choice |
| --- | --- |
| State management | Riverpod (with code generation) |
| Navigation | go\_router |
| Local database | Drift + SQLCipher (encrypted, offline-first) |
| Secure storage | flutter\_secure\_storage (Keychain / Keystore) for keys |
| Crypto | `cryptography` package: XChaCha20-Poly1305 for payloads, Argon2id for the passphrase |
| Networking | Dio + Retrofit; WebSocket (socket\_io\_client) for chat streaming and community |
| Auth | firebase\_auth or own JWT; Google + Apple sign-in packages |
| Notifications | firebase\_messaging + flutter\_local\_notifications |
| Lock and disguise | local\_auth (biometric), alternate app icons (iOS `setAlternateIconName`, Android activity-alias) |
| Speech | On-device speech\_to\_text; audio playback for TTS (v2) |
| Localization | flutter\_localizations + ARB files, RTL support |
| Purchases | RevenueCat or in\_app\_purchase (v2) |
| Glass UI | Custom `GlassSurface` widgets on `BackdropFilter` (section 10) |

App structure (feature-first): `lib/core` (theme, crypto, network, db), `lib/features/{onboarding, tracking, coach, sos, community, settings, lock}`, each with `data / domain / presentation`.

### 8.2 NestJS API

| Module | Responsibility | Key endpoints / events |
| --- | --- | --- |
| auth | Sign-up, login, Google/Apple, JWT + refresh, 18+ gate | `POST /auth/*` |
| users | Profile, coach settings, devices, export, delete | `GET/PATCH /me`, `POST /me/export`, `DELETE /me` |
| sync | Encrypted entries and items, conflict resolution (last-write-wins per record + version) | `POST /sync/push`, `GET /sync/pull?since=` |
| coach | Conversations, streaming replies, memory summaries, onboarding plan | `POST /coach/messages` (SSE or WS stream) |
| llm | Provider abstraction, prompt assembly, model routing, cost tracking | internal |
| safety | Crisis classifier, hotline lookup, trusted-contact flow | internal + `GET /safety/hotlines` |
| nudges | Scheduler, nudge generation, push sending, limits | Cloud Tasks handlers |
| community | Pseudonyms, posts, replies, reactions, (v2) circles, DMs | `/community/*`, WS gateway |
| moderation | AI classification, reports, queue, actions, audit | `/mod/*` (Moderator role) |
| billing | Store webhooks, entitlements (v2) | `/billing/webhooks/*` |
| admin | Hotline directory, rules, Moderator management | `/admin/*` |

Cross-cutting: class-validator DTOs, rate limiting (per user and per IP), request-body redaction in logs, OpenAPI docs, BullMQ (Redis) for background jobs, health checks.

### 8.3 Infrastructure (GCP)

- **Region:** europe-west1 (Belgium) for GDPR; all services in the same region.
- **Compute:** Cloud Run (API, autoscaling, min 1 instance to avoid cold starts on chat).
- **Data:** Cloud SQL for PostgreSQL 16 with automated backups and point-in-time recovery; Memorystore Redis.
- **Security:** Secret Manager, Cloud KMS, Cloud Armor (rate limits / WAF), IAM least privilege.
- **CI/CD:** GitHub Actions → Cloud Build → Cloud Run; Codemagic or Fastlane for app builds.
- **Monitoring:** Cloud Logging (redacted), Error Reporting, Sentry in the app (PII scrubbing on).

## 9. Data model

PostgreSQL holds three separate schemas so that a leak or bug in one area cannot expose another: `account`, `private` (encrypted user data) and `community`.

| Schema | Table | Key columns | Notes |
| --- | --- | --- | --- |
| account | users | id, email, auth\_provider, locale, birth\_year\_confirmed, created\_at, deleted\_at | Email encrypted |
| account | subscriptions | user\_id, platform, product\_id, status, expires\_at | From store webhooks |
| account | devices | id, user\_id, push\_token, platform, last\_seen | For FCM/APNs |
| account | key\_backups | user\_id, wrapped\_master\_key, kdf\_params | E2E key backup |
| private | profiles | user\_id, coach\_name, coach\_gender, voice\_id, tone, modes\[\], profile\_enc | profile\_enc = KMS-encrypted JSON |
| private | tracked\_items | id, user\_id, mode, category, payload\_enc, created\_at | Label, goal, motivation in payload\_enc (E2E) |
| private | entries | id, user\_id, item\_id, type (checkin/slip/urge/journal), occurred\_on, payload\_enc, updated\_at | Sync table, E2E |
| private | nudge\_schedules | user\_id, check\_in\_time, risky\_windows\[\], max\_per\_day, quiet\_start, quiet\_end, timezone | Plain metadata only |
| private | ai\_conversations | id, user\_id, kind (onboarding/coach/sos/crisis), created\_at |  |
| private | ai\_messages | id, conversation\_id, role, content\_enc, safety\_level, created\_at | Deleted after 90 days |
| private | ai\_memory | user\_id, summary\_enc, version, updated\_at | Rolling summary |
| private | trusted\_contacts | user\_id, name\_enc, channel, value\_enc, message\_enc |  |
| community | pseudonyms | id, display\_name, avatar\_seed, gender\_pref, locale, strikes, status | No user\_id here |
| community | identity\_links | pseudonym\_id, user\_id\_enc | Separate KMS key, audited access |
| community | circles | id, category, mode, language, same\_gender, member\_count | v2 |
| community | posts | id, pseudonym\_id, circle\_id?, visibility, body\_enc, cw, moderation\_state, created\_at |  |
| community | replies | id, post\_id, pseudonym\_id, body\_enc, moderation\_state |  |
| community | reactions | post\_id, pseudonym\_id, type |  |
| community | dm\_threads / dm\_messages | id, a\_pseudonym, b\_pseudonym, consent\_state / thread\_id, body\_enc, moderation\_state | v2 |
| community | reports | id, target\_type, target\_id, reporter\_pseudonym, reason, ai\_score, status, assigned\_to |  |
| community | moderation\_actions | id, moderator\_id, target, action, reason, created\_at | Audit log |
| system | hotlines | country\_code, name, phone, url, language | Maintained list |

Conventions: UUIDv7 primary keys, `created_at`/`updated_at` everywhere, soft delete then hard delete after 30 days, Prisma or TypeORM migrations (Prisma recommended for speed).

## 10. Design system: liquid glass

TurnWell uses frosted, translucent glass surfaces floating over a slowly moving gradient, with three themes the user can choose; by default, Dawn is used by day and Night Sanctuary by night.

### 10.1 Themes

| Token | Dawn (default day) | Night Sanctuary (default night) | Oasis |
| --- | --- | --- | --- |
| Background gradient | #2B2D6E → #6C5B9E → #F7B59B | #070B1A → #121A3A + aurora #3B2A6B | #0F766E → #5EB8A8 → #EADBC0 |
| Accent | Gold #F5B971 | Violet #A78BFA | Emerald #34D399 |
| Glass fill | White 18% | #0B1020 at 45% | White 22% |
| Glass border | White 35%, 1 px | White 12%, 1 px | White 40%, 1 px |
| Text primary | #FFFFFF | #E8EAF6 | #0B2B26 |
| Text secondary | White 72% | #E8EAF6 at 60% | #0B2B26 at 65% |
| Progress art | Rising sun | Stars lighting up | Growing garden |
| Release | v1 | v1 | v2 |

Slips use a neutral grey (#9CA3AF family) in every theme, never red. Red is reserved for destructive actions (delete, panic wipe) only.

### 10.2 Glass components (Flutter)

- `GlassSurface`: `BackdropFilter` blur 24–30 sigma, fill + 1 px border + subtle inner highlight on the top edge; corner radius 24 (cards), 32 (sheets), pill (buttons).
- `GlassCard`, `GlassButton` (primary = accent fill 85%), `GlassNavBar` (floating bottom bar, 5 tabs), `GlassSheet`, `GlassChip`, `GlassInput`.
- `MeshBackground`: animated gradient (slow, 20–40 s loop), paused when "Reduce motion" is on.
- `SOSButton`: floating, always visible on Home, accent glow.
- Performance: limit stacked blurs to 2 layers; use cached blur for lists; fall back to solid translucent fills on low-end Android.

### 10.3 Layout and type

- Bottom navigation: **Home** (today, check-in, SOS), **Coach**, **Progress**, **Community**, **Me**.
- Typeface: Inter (Latin) + IBM Plex Sans Arabic (Arabic). Sizes 34 / 22 / 17 / 15 / 13.
- Spacing scale 4 / 8 / 12 / 16 / 24 / 32. Touch targets at least 48 × 48.
- RTL: all layouts mirrored with `Directionality`; icons with direction flip.
- Accessibility: WCAG AA contrast on glass (text sits on a darker scrim when needed), Dynamic Type support, screen-reader labels.
- Haptics: light tap on check-in, soft success on urge passed.

### 10.4 Tone of UI copy

Warm, short, never judging. Examples: "Slipped today? That's okay. What happened?" — "You rode out 4 urges this week." — never "You failed" or "Streak lost".

## 11. Localization, business, legal and roadmap

### 11.1 Localization

- UI v1: English, Arabic (RTL), French, Spanish, via Flutter `intl` ARB files. Fallback English.
- Coach and moderation: any language Gemini handles; moderation quality is checked per language before opening circles in it.
- Circles are split by language; reach-out posts show the user's language first.
- Sins-mode Islamic terms kept in Arabic with translation (e.g. *tawbah* — repentance).

### 11.2 Monetization

| Plan | Includes | Price (to validate) |
| --- | --- | --- |
| Free | Tracking, journal, SOS, community, 15 AI messages/day, nudges, Dawn + Night themes | 0 |
| Premium monthly | Unlimited AI, voice coach, weekly reflection, deep insights, all themes | \~4.99 USD, with regional pricing |
| Premium yearly | Same | \~39.99 USD, with regional pricing |

Principles: crisis help, SOS and community are always free. No ads. Regional pricing for MENA and other lower-income markets.

### 11.3 Legal checklist

- [ ] Choose the publishing legal entity (blocks store accounts, payouts and the privacy policy)
- [ ] Privacy policy + terms written for GDPR (EU users), Algeria Law 18-07, and US state privacy laws
- [ ] Data processing agreement with Google Cloud / Vertex AI; confirm data residency (EU region)
- [ ] App Store health / mental-wellness review notes and the "not a medical service" disclaimer
- [ ] Trademark search for "TurnWell" in the EU, US and target markets
- [ ] Hotline directory sources and review cadence
- [ ] Moderator guidelines and legal-request process

### 11.4 Success metrics

| Metric | Target at 3 months after launch |
| --- | --- |
| Onboarding completion (sign-up → plan created) | 70% |
| Day-7 retention | 35% |
| Day-30 retention | 20% |
| Users with 3+ check-ins per week | 50% of actives |
| Urges logged as "passed" | 60% |
| Self-reported improvement (monthly survey) | 60% "better" |
| Moderation: harmful content seen by others | < 0.1% of posts |
| AI cost per monthly active user | < 0.15 USD |

### 11.5 Roadmap

- **v1 (MVP):** onboarding conversation + plan, tracking, journal, coach chat, proactive nudges, SOS, crisis protocol, reach-out posts with AI moderation, disguise + PIN, Dawn + Night themes, 4 languages. Detailed in the [companion build plan](./MVP.md).
- **v2:** circles, DMs, accountability partners, subscriptions, voice coach, weekly reflection, replacement habits, Sins-mode positive practices and prayer times, widgets, Oasis theme.
- **v3:** verified mentors, learning modules, web app, more languages, B2B partnerships (NGOs, associations).

### 11.6 Open items

- Publishing legal entity: TBD.
- Vertex AI zero-data-retention / abuse-monitoring logging settings to confirm before launch.
- Final prices after first user tests.
- Voice provider for text-to-speech (Google Cloud TTS vs others) and voices per language.

### Sources

- [Gemini API pricing (2026)](https://www.morphllm.com/gemini-api-pricing) — Flash-Lite $0.10 / $0.40, Flash $0.30 / $2.50 per million tokens
- [Gemini API pricing guide](https://developer.puter.com/tutorials/gemini-api-pricing/) — free tier, batch discount, audio pricing
- [Official Gemini pricing page](https://ai.google.dev/gemini-api/docs/pricing)
