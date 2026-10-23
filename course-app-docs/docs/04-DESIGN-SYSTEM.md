# 04 — Design System

## Direction
Focused, calm, and Indian-context friendly. Deep ink-blue for trust and focus, with a single saffron accent reserved for progress and the main purchase action. Hindi text must look as good as English.
The one memorable element: **the progress ribbon** — a thin saffron bar running along the top of every course card and the player, so learners always see how far they are.

## Tokens (implement in `lib/app/theme/tokens.dart`; never use raw values elsewhere)
### Colour
| Token | Hex | Use |
|-------|-----|-----|
| `ink` | #1E2A5A | Primary, app bar, headings |
| `paper` | #F7F8FB | App background |
| `surface` | #FFFFFF | Cards, sheets |
| `saffron` | #F2A33A | Accent: progress, Buy button |
| `slate` | #3C4256 | Body text |
| `mist` | #E3E6EF | Borders, dividers |
| `success` | #2F9E7A | Success states |
| `danger` | #C8453B | Errors |
Dark theme: `paper` → #10142A, `surface` → #1A2040, text → #E8EAF4; keep `saffron` the same. Check AA contrast for every pair.

### Typography
- Headings: **Sora** (600/700). Body & UI: **Hind** (400/500/600) — supports Latin and Devanagari.
- Scale (sp): display 28, title 22, heading 18, body 16, label 14, caption 12. Body line-height 1.5; Hindi line-height 1.6.
- Sentence case everywhere. No all-caps labels.

### Spacing / shape / elevation
- Spacing scale: 4, 8, 12, 16, 24, 32. Screen padding 16.
- Radius: 8 for inputs and chips, 12 for cards, 24 for bottom sheets. Different radii by role, not one value for everything.
- Elevation: cards use a 1px `mist` border, no shadow. Only sheets and dialogs get shadow.
- Min touch target 48×48dp.

## Core components (build once in `core/widgets/`)
`AppButton` (primary saffron-on-ink, secondary outline, text), `AppTextField`, `CourseCard` (thumbnail, title, rating, price, progress ribbon), `PriceTag` (MRP struck through + price), `LessonTile`, `SectionHeader`, `RatingStars`, `EmptyState`, `ErrorState` (message + Retry), `SkeletonLoader`, `AppSnackbar`.
Every list screen has: loading skeleton, empty state, error state with retry.

## Screens & layout
- **Bottom nav (4):** Home · My courses · Search · Profile.
- **Home:** search bar, featured carousel, categories chips, "Continue learning" strip (only if enrolled).
- **Course detail:** thumbnail/preview → title, rating, price → sticky bottom bar with Buy button → syllabus accordion → reviews.
- **Player:** video on top (16:9), lesson list below, notes tab; landscape = fullscreen.
- Left-align text. Max ~60 characters per line on tablets.

## Motion
Respond to actions only (button press, accordion open, progress updating). One entrance moment: progress ribbon fills on My courses. No decorative fade-ins. Respect "reduce motion".

## 3D hero
The play-cube is the signature 3D object: top face saffron, side faces blue (#4A66CC, #2D3F8F), play mark on top, page lines on the side, soft ground shadow. Four moments and rules are in docs/08-3D-ANIMATION.md. Figma file has a "3D animation spec" board and a "Home with 3D hero" variant.

## Copy rules
- Say what happens: "Buy for ₹499", "Resume lesson 4", not "Submit".
- Same action name across the flow: Buy → "Payment successful. Start learning".
- Errors: what went wrong + what to do. "Payment didn't go through. Try again or use another method." No apologies, no vague "Something went wrong".
- Empty states invite action: "No courses yet. Browse courses".
- All strings in `l10n/app_en.arb` and `app_hi.arb`. No hardcoded text in widgets.

## Assets
App icon: simple ink-blue rounded square with a saffron play-triangle forming an open book. Provide adaptive icon (foreground/background). Illustrations: flat, 2 colours max from tokens.
