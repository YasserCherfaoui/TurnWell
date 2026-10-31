# TurnWell design kit for Antigravity

Everything from the TurnWell UI design (20 boards) packaged so an Antigravity agent can rebuild it
in Flutter exactly.

## What's inside
| Path | What it is |
|---|---|
| `design/screens/*.html` | Each board as a standalone HTML file (opens in any browser; the timer, stars and calendar run live) |
| `design/png/*.png` | Reference render of each screen at 390×844 @2x (Main at 1600 wide) |
| `flutter_starter/lib/design/tw_css.dart` | Exact CSS→Flutter primitives: CSS-angle gradients, `circle at X% Y%` radials, Glass surface with inner specular highlight, aurora blobs |
| `flutter_starter/lib/design/tw_tokens.dart` | All colors, gradients, category/SOS colors, orbs, radii, type styles from the design system |
| `flutter_starter/assets/icons/` | 87 UI icons as SVG + `_manifest.json` (where each is used) |
| `AGENTS.md`, `.agents/rules/turnwell-design-fidelity.md` | Rules Antigravity loads automatically |
| `PROMPTS.md` | The prompts to paste, in order |
| `raw/` | Original design files, untouched (backup) |

## Setup (5 minutes)
1. Create the Flutter project: `flutter create turnwell` (or use your existing one).
2. Copy into the project root:
   - `AGENTS.md` and the `.agents/` folder
   - the `design/` folder
   - `flutter_starter/lib/design/` → `lib/design/`
   - `flutter_starter/assets/` → `assets/`
3. `flutter pub add flutter_svg google_fonts go_router`
4. Open the folder in Antigravity → Customizations → Rules: check the TurnWell rule shows up
   (if your version uses `.agent/rules/` instead of `.agents/rules/`, rename the folder).
5. Paste the prompts from `PROMPTS.md`, one at a time.

## Getting a truly identical result
- Test on a 390×844 device (iPhone 13/14/15) — that is the design frame.
- Make the agent compare its screenshot to the PNG after every screen (prompt 2 already does).
- Antigravity's browser agent can open `design/screens/*.html` directly if it needs to inspect an element.
- Oasis theme is not designed yet; the agent is told not to invent it.
