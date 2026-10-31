# 04 — Design system: Liquid Glass

Frosted, translucent glass surfaces float over a slowly moving gradient. Calm, hopeful, never clinical.
Visual reference: `diagrams/mockups-dawn.png` and `diagrams/mockups-night.png`.

## 1. Themes

| Token | Dawn (`DAWN`, default by day) | Night Sanctuary (`NIGHT_SANCTUARY`, default by night) | Oasis (`OASIS`, v2) |
| --- | --- | --- | --- |
| `bgGradient` | `#2B2D6E → #6C5B9E → #F7B59B` (top → bottom) | `#070B1A → #121A3A`, aurora blob `#3B2A6B` | `#0F766E → #5EB8A8 → #EADBC0` |
| `accent` | `#F5B971` (gold) | `#A78BFA` (violet) | `#34D399` (emerald) |
| `onAccent` | `#2B1A05` | `#14082E` | `#04241A` |
| `glassFill` | `#FFFFFF` @ 18% | `#0B1020` @ 45% | `#FFFFFF` @ 22% |
| `glassBorder` | `#FFFFFF` @ 35% | `#FFFFFF` @ 12% | `#FFFFFF` @ 40% |
| `glassHighlight` (top inner edge) | `#FFFFFF` @ 55% | `#FFFFFF` @ 18% | `#FFFFFF` @ 60% |
| `textPrimary` | `#FFFFFF` | `#E8EAF6` | `#0B2B26` |
| `textSecondary` | `#FFFFFF` @ 72% | `#E8EAF6` @ 60% | `#0B2B26` @ 65% |
| `neutralSlip` | `#9CA3AF` | `#6B7280` | `#9CA3AF` |
| `success` | `#86EFAC` | `#6EE7B7` | `#065F46` |
| `danger` (destructive only) | `#F87171` | `#F87171` | `#B91C1C` |
| Progress art | Rising sun | Stars lighting up | Growing garden |

`AUTO` theme: Dawn from 06:00 to 18:59 local, Night Sanctuary from 19:00 to 05:59 (follows the device clock, re-evaluated on resume).

**Rule:** slips are always `neutralSlip`. `danger` is only for destructive actions (delete account, panic wipe).

## 2. Typography

| Style | Latin (Inter) | Arabic (IBM Plex Sans Arabic) | Use |
| --- | --- | --- | --- |
| `display` | 34 / 600 / -0.5 | 32 / 600 | Big numbers (78% clean) |
| `title` | 22 / 600 | 22 / 600 | Screen titles |
| `headline` | 17 / 600 | 17 / 600 | Card titles |
| `body` | 15 / 400 / 1.45 lh | 16 / 400 / 1.6 lh | Text, chat |
| `caption` | 13 / 400 | 13 / 400 | Meta, hints |

Support Dynamic Type up to 200%: cards grow in height, never truncate key text.

## 3. Spacing, radius, elevation

- Spacing scale: `4, 8, 12, 16, 24, 32, 48`. Screen padding 16 (phones), 24 (tablets).
- Radius: chips/buttons pill (999), cards 24, sheets 32, inputs 16.
- No drop shadows. Depth comes from blur + border + highlight. Max **2 stacked blur layers** on screen.
- Touch targets ≥ 48×48.

## 4. Core components (Flutter, `lib/core/theme/`)

```dart
// lib/core/theme/turnwell_tokens.dart
@immutable
class TwTokens extends ThemeExtension<TwTokens> {
  const TwTokens({
    required this.bgGradient,
    required this.accent,
    required this.onAccent,
    required this.glassFill,
    required this.glassBorder,
    required this.glassHighlight,
    required this.textPrimary,
    required this.textSecondary,
    required this.neutralSlip,
    required this.success,
    required this.danger,
    required this.blurSigma,
  });

  final List<Color> bgGradient;
  final Color accent, onAccent, glassFill, glassBorder, glassHighlight;
  final Color textPrimary, textSecondary, neutralSlip, success, danger;
  final double blurSigma;

  static const dawn = TwTokens(
    bgGradient: [Color(0xFF2B2D6E), Color(0xFF6C5B9E), Color(0xFFF7B59B)],
    accent: Color(0xFFF5B971),
    onAccent: Color(0xFF2B1A05),
    glassFill: Color(0x2EFFFFFF), // 18%
    glassBorder: Color(0x59FFFFFF), // 35%
    glassHighlight: Color(0x8CFFFFFF), // 55%
    textPrimary: Color(0xFFFFFFFF),
    textSecondary: Color(0xB8FFFFFF), // 72%
    neutralSlip: Color(0xFF9CA3AF),
    success: Color(0xFF86EFAC),
    danger: Color(0xFFF87171),
    blurSigma: 24,
  );

  static const nightSanctuary = TwTokens(
    bgGradient: [Color(0xFF070B1A), Color(0xFF121A3A)],
    accent: Color(0xFFA78BFA),
    onAccent: Color(0xFF14082E),
    glassFill: Color(0x730B1020), // 45%
    glassBorder: Color(0x1FFFFFFF), // 12%
    glassHighlight: Color(0x2EFFFFFF), // 18%
    textPrimary: Color(0xFFE8EAF6),
    textSecondary: Color(0x99E8EAF6), // 60%
    neutralSlip: Color(0xFF6B7280),
    success: Color(0xFF6EE7B7),
    danger: Color(0xFFF87171),
    blurSigma: 30,
  );

  @override
  TwTokens copyWith() => this;

  @override
  TwTokens lerp(ThemeExtension<TwTokens>? other, double t) => t < 0.5 ? this : other as TwTokens;
}
```

```dart
// lib/core/theme/glass_surface.dart
class GlassSurface extends ConsumerWidget {
  const GlassSurface({super.key, required this.child, this.radius = 24, this.padding = const EdgeInsets.all(16)});
  final Widget child;
  final double radius;
  final EdgeInsetsGeometry padding;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final t = Theme.of(context).extension<TwTokens>()!;
    final lowEnd = ref.watch(deviceClassProvider) == DeviceClass.low; // core/device: RAM + GPU heuristic
    final body = Container(
      padding: padding,
      decoration: BoxDecoration(
        color: lowEnd ? t.glassFill.withOpacity(0.75) : t.glassFill,
        borderRadius: BorderRadius.circular(radius),
        border: Border.all(color: t.glassBorder, width: 1),
        gradient: LinearGradient( // inner top highlight
          begin: Alignment.topCenter,
          end: Alignment.center,
          colors: [t.glassHighlight.withOpacity(0.25), Colors.transparent],
        ),
      ),
      child: child,
    );
    if (lowEnd) return ClipRRect(borderRadius: BorderRadius.circular(radius), child: body);
    return ClipRRect(
      borderRadius: BorderRadius.circular(radius),
      child: BackdropFilter(filter: ImageFilter.blur(sigmaX: t.blurSigma, sigmaY: t.blurSigma), child: body),
    );
  }
}
```

| Component | Spec |
| --- | --- |
| `MeshBackground` | 3–4 soft radial blobs moving on a 30 s loop (`AnimationController`), colours from `bgGradient`. Static when `MediaQuery.disableAnimations`. |
| `GlassCard` | `GlassSurface` + optional header row (icon, title, trailing action) |
| `GlassButton` | Primary: accent fill @ 85%, `onAccent` text, pill, height 52. Secondary: glass. Tertiary: text only. |
| `GlassNavBar` | Floating, 16 from bottom/sides, radius 28, 5 icons + labels, active = accent pill behind icon |
| `GlassSheet` | Bottom sheet, radius 32 top, drag handle, blur |
| `GlassChip` | Pill, selectable (accent border + 12% accent fill when selected) |
| `GlassInput` | Radius 16, glass fill, 1 px border → accent on focus |
| `SosButton` | 64×64 circle, accent fill, soft glow (`BoxShadow` with accent @ 40%, blur 24 — the only shadow allowed), label "SOS", floating bottom-right above nav bar (bottom-left in RTL) |
| `ProgressRing` | Arc showing % clean days, accent stroke 10, track `glassBorder` |
| `MoodPicker` | 5 abstract faces/circles, no emoji |
| `CoachBubble` / `UserBubble` | Coach: glass left-aligned with avatar; user: accent @ 22% right-aligned (mirrored in RTL) |
| `CrisisCard` | Full-width, solid (not glass) high-contrast card, never blurred — see M09 |

## 5. Screen inventory and routes

| Route | Screen | Module |
| --- | --- | --- |
| `/splash` | Splash + language detect | M16 |
| `/welcome` | Privacy promise (3 points) | M01 |
| `/auth/sign-up`, `/auth/sign-in`, `/auth/verify-email` | Auth | M01 |
| `/lock/setup`, `/lock` | PIN setup / unlock | M02 |
| `/onboarding/mode` | Mode choice | M03 |
| `/onboarding/coach` | Coach setup | M03 |
| `/onboarding/chat` | Onboarding conversation | M03 |
| `/onboarding/plan` | Plan review | M03 |
| `/onboarding/safety` | Trusted contact + recovery passphrase | M09 / M06 |
| `/home` | Home (today) | M04 |
| `/home/check-in` (sheet) | Check-in | M04 |
| `/home/slip` (sheet) | Slip log | M04 |
| `/coach`, `/coach/:conversationId` | Coach chat | M07 |
| `/sos` (full-screen overlay) | SOS flow | M10 |
| `/progress`, `/progress/item/:id` | Progress | M04 |
| `/community`, `/community/post/:id`, `/community/compose` | Community | M12 |
| `/me`, `/me/journal`, `/me/journal/:id` | Me, journal | M05 |
| `/me/settings/*` | Settings | M14 |
| `/me/privacy/*` | Export, delete, panic wipe, disguise | M17 / M02 |
| `/crisis` (full-screen) | Crisis card | M09 |
| `/decoy` | Decoy to-do screen | M02 |

## 6. Home wireframe (reference layout)

```
┌──────────────────────────────────────┐
│  Good evening                    (⚙) │  ← title, settings icon
│  Thursday, 20 November               │
│ ╭──────────────────────────────────╮ │
│ │  ◯ 78%   clean days · 30 days    │ │  ← GlassCard + ProgressRing
│ │  ↗ up from 64% last month        │ │
│ ╰──────────────────────────────────╯ │
│ ╭──────────────────────────────────╮ │
│ │ Today's check-in          21:30  │ │
│ │ ● Pornography       [ Check in ] │ │  ← one row per item
│ │ ● Smoking           ✓ Clean      │ │
│ ╰──────────────────────────────────╯ │
│ ╭──────────────────────────────────╮ │
│ │ (N) Noor                         │ │  ← latest coach nudge preview
│ │ "Evenings have been hard. Want   │ │
│ │  to plan tonight together?"      │ │
│ │                    [ Reply → ]   │ │
│ ╰──────────────────────────────────╯ │
│ ╭──────────────╮ ╭─────────────────╮ │
│ │ 4 urges      │ │ Plan for today  │ │
│ │ passed ✓     │ │ 3 of 4 done     │ │
│ ╰──────────────╯ ╰─────────────────╯ │
│                               ╭────╮ │
│                               │SOS │ │  ← SosButton (glow)
│                               ╰────╯ │
│ ╭──────────────────────────────────╮ │
│ │ ⌂ Home  ✦ Coach  ◔ Progress  ◎ … │ │  ← GlassNavBar
│ ╰──────────────────────────────────╯ │
└──────────────────────────────────────┘
```

## 7. Motion

| Interaction | Animation |
| --- | --- |
| Screen transition | Fade + 8 px slide, 250 ms, `Curves.easeOutCubic` |
| Sheet open | Spring (stiffness 300, damping 30) |
| Check-in "clean" | Ring fills + light haptic + small sunrise/star sparkle (300 ms) |
| Urge passed | Soft success haptic + progress art step |
| Coach typing | 3-dot shimmer in a glass bubble |
| Reduce motion | All of the above become 150 ms cross-fades; background static |

## 8. RTL and accessibility

- Wrap the app in `Directionality` from locale; use `EdgeInsetsDirectional`, `AlignmentDirectional` everywhere.
- Mirror directional icons (back, reply, arrows); do not mirror the progress ring or clocks.
- Text on glass must reach WCAG AA (4.5:1). If the background behind a card is too light, `GlassSurface` adds a
  `#000000 @ 12%` scrim (Dawn) automatically when contrast check fails (pre-computed per theme).
- Every icon button has a `Semantics(label: ...)`. Progress ring announces "78 percent clean days in the last 30 days".

## 9. Copy tone

Warm, short, never judging. ✓ "Slipped today? That's okay. What happened?" ✓ "You rode out 4 urges this week."
✗ "You failed" ✗ "Streak lost" ✗ "Bad day!".
