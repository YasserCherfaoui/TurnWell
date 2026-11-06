# Mobile Walkthroughs

Step-by-step educational documentation of the TurnWell Flutter mobile app, state architecture, Liquid Glass design system, and feature implementations.

## Chapters

| # | Chapter | Key Topics Covered |
| :--- | :--- | :--- |
| **0001** | [0001 — Mobile Foundations, Liquid Glass Design System & Auth UI](./0001-setup-design-system-and-auth.md) | Flutter project setup, `TwTokens` (Dawn & Night Sanctuary), `GlassSurface`, `MeshBackground`, Riverpod 3 `Notifier<AuthState>`, `Dio` client with single-flight refresh lock, `GoRouter` guard pipeline, M01 auth screens (Welcome, Sign Up, Verify, Sign In, Age Gate, Reset), RTL Arabic support, and widget tests. |
| **0002** | [0002 — App Lock & Disguise System (M02)](./0002-app-lock-and-disguise.md) | Mandatory 6-digit PIN lock, weak PIN validator, Argon2id hasher, timing-safe dual verification, biometric unlock (`local_auth`), brute-force throttle back-off schedule, auto-lock lifecycle observer, app switcher privacy shield, harmless decoy to-dos (`/decoy`), disguise alternate icons, non-mirrored LTR keypad, and unit/widget test suites. |


