# Mobile Walkthroughs

Step-by-step educational documentation of the TurnWell Flutter mobile app, state architecture, Liquid Glass design system, and feature implementations.

## Chapters

| # | Chapter | Key Topics Covered |
| :--- | :--- | :--- |
| **0001** | [0001 — Mobile Foundations, Liquid Glass Design System & Auth UI](./0001-setup-design-system-and-auth.md) | Flutter project setup, `TwTokens` (Dawn & Night Sanctuary), `GlassSurface`, `MeshBackground`, Riverpod 3 `Notifier<AuthState>`, `Dio` client with single-flight refresh lock, `GoRouter` guard pipeline, M01 auth screens (Welcome, Sign Up, Verify, Sign In, Age Gate, Reset), RTL Arabic support, and widget tests. |
| **0002** | [0002 — App Lock & Disguise System (M02)](./0002-app-lock-and-disguise.md) | Mandatory 6-digit PIN lock, weak PIN validator, Argon2id hasher, timing-safe dual verification, biometric unlock (`local_auth`), brute-force throttle back-off schedule, auto-lock lifecycle observer, app switcher privacy shield, harmless decoy to-dos (`/decoy`), disguise alternate icons, non-mirrored LTR keypad, and unit/widget test suites. |
| **0003** | [0003 — Multi-Language Architecture & ARB Expansion (M16)](./0003-internationalization-and-arb.md) | Official Flutter `gen-l10n` toolchain, 4 language catalogs (EN, AR, FR, ES), Arabic 6-case ICU plurals, gendered coach string selection, Sins mode faith term glossing, `LocaleNotifier` device vs explicit preference resolution, liquid glass language picker sheet, backend `I18nService` parity, and ARB key parity tests. |
| **0004** | [0004 — Mode Choice & Coach Setup Screens (M03)](./0004-mode-choice-and-coach-setup.md) | Mode choice (`/onboarding/mode` for Sins/Habits/Both + Islamic notice), Coach setup (`/onboarding/coach` for gender, voice preview, tone, localized default names, non-authority blocklist), `PUT /onboarding/setup` API sync, GoRouter onboarding gate, EN/AR/FR/ES ARB strings, and widget/unit tests. |
| **0005** | [0005 — Floating Glass Navigation Shell, SOS Button & Me Hub](./0005-bottom-nav-shell-and-me-hub.md) | Floating `GlassNavBar` (5 tabs: Home, Coach, Progress, Community, Me), glowing 64×64 `SosButton`, `StatefulShellRoute.indexedStack` in `GoRouter`, `MeScreen` privacy/disguise/language hub, blur-stack compliance, RTL mirroring tests, and 4-language ARB parity. |





