# Prompts to paste into Antigravity (in this order)

Use Agent mode, Planning on, the strongest Gemini model available. One prompt per conversation
step; review the result before the next one.

---

## Prompt 1 — Setup & shared design system
```
Read AGENTS.md and .agents/rules/turnwell-design-fidelity.md fully, then open
design/png/Main.png and design/screens/Main.html (the design system board).

1. Make sure pubspec.yaml has: flutter_svg, google_fonts, go_router. Register assets/icons/
   and assets/illustrations/ as assets.
2. Using lib/design/tw_tokens.dart and lib/design/tw_css.dart (do not rewrite them), build these
   shared widgets in lib/design/widgets/: AuroraBackground (dawn + night variants, incl. star
   field), GlassCard, GlassPill, IconTile, PrimaryButton (dawn + night), GlassButton,
   SelectChip, GlassNavBar (5 tabs, active pill), SosButton (dawn + night), CoachOrb,
   ChatBubble (coach + user).
3. Create lib/dev/design_gallery.dart that shows all of them on Dawn and Night backgrounds,
   laid out like the Components panels in Main.png.
4. Run it, screenshot it, compare with design/png/Main.png and fix differences. Report what still differs.
```

## Prompt 2 — One screen at a time (repeat for each screen)
```
Implement the screen "Home-Dawn".
Source: design/screens/Home-Dawn.html (exact CSS) and design/png/Home-Dawn.png (target look).
Follow .agents/rules/turnwell-design-fidelity.md. Reuse lib/design/widgets. Copy every number
from the HTML. Icons from assets/icons via _manifest.json. Copy the sunrise SVG to
assets/illustrations/ and render it with flutter_svg.
Put it in lib/features/home/home_screen.dart with fake data matching the design text.
When done: run it at 390×844, screenshot, compare side by side with the PNG, list the
differences, fix them, and show me the final side-by-side.
```

Screen order (matches the design canvas):
1. Onboarding: 01-Welcome, 02-SignUp, 03-LockSetup, 04-Mode, 05-CoachSetup, 06-OnboardingChat,
   07-PlanReview, 08-Safety
2. Home & check-in: Home-Dawn, Home-Night, CheckIn (bottom sheet), SlipLog (bottom sheet), Progress
3. Coach, SOS & crisis: Coach-Dawn, Coach-Night, SOS-Tools, SOS-RideUrge, SOS-Outcome, Crisis

Tip: for the Night version of a screen that already exists in Dawn (Home-Night, Coach-Night),
say: "Home-Night is the same screen as Home-Dawn in the Night Sanctuary theme — make the theme
switch drive the differences instead of duplicating the screen."

## Prompt 3 — Navigation
```
Wire all implemented screens with go_router following the links in design/screens/*.html
(every <a href="X.html"> is a navigation to screen X). CheckIn and SlipLog are modal bottom
sheets; the bottom GlassNavBar is a ShellRoute. The SOS button opens SOS-Tools as a full-screen route.
```

## Prompt 4 — Golden tests (lock the look)
```
Add golden tests at 390×844 for every screen with the Sora/Inter fonts loaded, so any future
change that breaks the design fails CI.
```
