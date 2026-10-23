# AGENTS.md — Course App (Flutter + Supabase)

You are the coding agent for **LearnApp** (working name), an Android course-selling app.
**Before writing ANY code, read these files in order, fully:**

1. docs/01-PRD.md — what we build and what we do NOT build
2. docs/02-ARCHITECTURE.md — structure, stack, data flow
3. docs/03-DATABASE.md — schema, RLS, migrations
4. docs/04-DESIGN-SYSTEM.md — tokens, components, copy rules
5. docs/05-SECURITY.md — non-negotiable security rules
6. docs/06-CODE-STYLE.md — naming, structure, patterns
7. docs/07-TESTING.md — what tests every change needs
8. docs/08-3D-ANIMATION.md — 3D animation moments, Rive setup, fallbacks, performance budget

If two docs conflict: SECURITY > DATABASE > ARCHITECTURE > PRD > CODE-STYLE > DESIGN-SYSTEM. Stop and ask the human instead of guessing.

## Stack (fixed — do not swap without asking)
- App: Flutter (stable), Dart 3, Android only for now
- State: flutter_riverpod | Routing: go_router | Models: freezed + json_serializable
- Backend: Supabase (Postgres, Auth, Storage, Edge Functions in TypeScript/Deno)
- Video: Bunny Stream (HLS, token-signed URLs) | Payments: Razorpay (behind `PaymentGateway` interface)
- Animation: Rive (.riv) for the 3D play-cube, with static fallback
- Push: Firebase Cloud Messaging | Crash reports: Firebase Crashlytics

## How you work
- Build in the phases below. One phase at a time. One feature = one small PR-sized change.
- For each task: (1) restate the goal in 2 lines, (2) list files you will touch, (3) implement, (4) add tests, (5) run checks, (6) summarise.
- Never invent packages, API fields, or table columns. If it is not in the docs, ask.
- Never hardcode secrets, keys, or URLs. Use `--dart-define` (app) and Supabase secrets (server).
- Never put business rules in widgets. Widgets render state; logic lives in `domain/` and `application/`.
- Prices are integers in paise. Never use double for money.

## Commands (must pass before you say "done")
```
dart format --set-exit-if-changed .
flutter analyze
flutter test
supabase db reset && supabase test db      # pgTAP RLS tests
deno test supabase/functions               # Edge Function tests
```

## Phases
- **P0 Setup:** repo, flavors (dev/prod), CI, theme tokens, router shell, Supabase project, env handling.
- **P1 Auth + Profile:** Google sign-in, email OTP, profile row, sign-out, session restore.
- **P2 Catalog:** home, categories, search, course detail, free-preview lessons.
- **P3 Purchase:** orders, Razorpay checkout, webhook verification, enrollment, coupons.
- **P4 Learning:** My Courses, secure HLS player, progress tracking, resume, offline-safe states.
- **P5 Engagement:** reviews, push notifications, certificates (PDF), wishlist, 3D animations M1–M4 (docs/08; human supplies .riv files, you build `Hero3D` + fallbacks).
- **P6 Admin + Release:** admin panel (web), analytics, Play Console assets, privacy policy, release build.
At the end of each phase: stop, summarise, wait for human review.

## Definition of Done
- Matches PRD acceptance criteria for that feature
- Uses only design tokens (no raw colours / font sizes / magic spacing)
- Loading, empty, error states all implemented
- Tests added per docs/07-TESTING.md; all commands above pass
- No secrets, no `print`, no TODO without a linked issue

## Never
- Never expose a video URL or lesson content without an enrollment check on the server
- Never trust the client for price, payment status, or enrollment
- Never disable RLS, never use the service-role key in the app
- Never create or fake `.riv` animation files; use the fallback image until the designer supplies them
- Never add a dependency without stating why and checking it is maintained
- Never rewrite unrelated files while doing a task
