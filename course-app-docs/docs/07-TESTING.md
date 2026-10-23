# 07 — Testing

## Pyramid
| Level | Tool | What | Target |
|-------|------|------|--------|
| Unit | `flutter_test`, `mocktail` | domain use cases, repositories (mock Supabase), notifiers, price/coupon formatting | ≥ 80% on `domain/` + `application/` |
| Widget | `flutter_test` + golden tests | each core component and each screen state (loading, empty, error, data) | every screen has all 4 states tested |
| Integration | `integration_test` | login → browse → buy (test mode) → play → progress; runs on emulator in CI | critical flows only |
| DB / RLS | pgTAP via `supabase test db` | every policy: allowed and denied cases | 100% of tables |
| Edge Functions | `deno test` | create-order pricing, webhook signature + idempotency, get-playback-url access rules | 100% of branches for payment + access |

## Must-have tests (never skip)
1. Learner A cannot read learner B's orders, progress, enrollments.
2. Client cannot insert into `orders`, `payments`, `enrollments`.
3. `get-playback-url` denies non-enrolled, allows enrolled, allows free preview, URL expires.
4. Webhook with bad signature → 400; replayed webhook → no duplicate enrollment.
5. Coupon: expired, maxed out, percent and flat discounts, never below ₹0.
6. Price is always taken from DB, not from client input.
7. Progress: 90% marks complete; resume position restores.

## 3D animation tests
- Widget test: `Hero3D` shows fallback image when `disableAnimations` is true, when asset fails, and on low-end flag.
- Widget test: play-once moments do not replay after rebuild.
- Manual: profile on a 3 GB RAM device; check no jank on Home scroll and cold start < 3 s.

## Conventions
- Test files mirror `lib/` paths: `lib/features/x/y.dart` → `test/features/x/y_test.dart`.
- Name: `should <behaviour> when <condition>`. Arrange–Act–Assert. No real network in unit/widget tests.
- Use fakes for repositories in widget tests; seed data from `test/fixtures/`.
- Bug fix = first a failing test, then the fix.

## CI (GitHub Actions) — blocks merge if any fails
`dart format` → `flutter analyze` → `flutter test --coverage` → `supabase db reset` + `supabase test db` → `deno test` → `gitleaks` → build debug APK.

## Manual QA before release
Low-end device (2–3 GB RAM), slow 3G, airplane mode mid-payment, app killed during payment, token expiry mid-video, Hindi + large font scale, dark mode, screen-recording blocked in player.
