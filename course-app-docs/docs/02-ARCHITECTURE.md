# 02 — Architecture

## Overview
```
Flutter app ──HTTPS──> Supabase (Auth, Postgres+RLS, Storage, Edge Functions)
     │                         │
     │                         ├── Razorpay (create order, webhook)
     │                         ├── Bunny Stream (signed HLS URLs)
     │                         └── FCM (push)
     └── plays HLS directly from Bunny CDN using the signed URL
```
The app never talks to Razorpay secrets or Bunny API keys. Anything privileged runs in Edge Functions.

## App structure (feature-first, clean layering)
```
lib/
  main.dart / main_dev.dart / main_prod.dart
  app/            # app widget, router, theme, env, l10n
  core/           # shared: errors (Result/AppException), network, storage, utils, widgets
  features/
    auth/
      data/         # repositories impl, DTOs, supabase calls
      domain/       # entities, repository interfaces, use cases
      application/  # riverpod notifiers/providers
      presentation/ # screens, widgets
    catalog/  purchase/  learning/  profile/  reviews/  notifications/
test/            # mirrors lib/
supabase/
  migrations/  functions/  tests/  seed.sql
docs/
```
Dependency rule: presentation → application → domain ← data. Domain imports nothing from Flutter or Supabase.

## State & data
- Riverpod `AsyncNotifier` per screen/feature. UI watches providers only.
- Repositories return `Result<T>` (success | AppFailure); no raw exceptions reach UI.
- Caching: in-memory + `shared_preferences`/Drift for catalog list (stale-while-revalidate). Auth tokens in `flutter_secure_storage` (handled by supabase_flutter).
- Navigation: go_router, auth redirect guard, deep links `learnapp://course/{id}`.

## Edge Functions (TypeScript)
| Function | Purpose |
|----------|---------|
| `create-order` | Validate course + coupon, compute price on server, create Razorpay order + `orders` row |
| `razorpay-webhook` | Verify signature, mark payment captured, create `enrollments` (idempotent) |
| `get-playback-url` | Check enrollment or free preview, return Bunny signed URL (TTL ≤ 10 min) |
| `register-device` | Enforce max 2 active devices per user |
| `delete-account` | Delete user data per policy |
| `send-push` | Triggered by DB webhook on new lesson / purchase |

## Environments
`dev` and `prod` Supabase projects. Flutter flavors `dev`/`prod`. Config via `--dart-define-from-file=env/dev.json` (anon key + URL only; never service-role).

## Key packages (pinned in pubspec; justify any addition)
flutter_riverpod, go_router, supabase_flutter, freezed, json_serializable, google_sign_in, razorpay_flutter, video_player + chewie (or better_player_plus), rive, firebase_messaging, firebase_crashlytics, flutter_secure_storage, cached_network_image, flutter_localizations/intl, mocktail.

## Release
Android App Bundle (.aab), Play App Signing, R8 minify + obfuscation (`--obfuscate --split-debug-info`), CI via GitHub Actions.
