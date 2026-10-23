# 05 — Security (non-negotiable)

## Secrets
- Allowed in app: Supabase URL + anon key, Razorpay **key id** only.
- Server only (Supabase secrets): service-role key, Razorpay key secret + webhook secret, Bunny API key + token-auth key, FCM service account.
- `.env`, `*.jks`, `key.properties`, `google-services.json` for prod are git-ignored. Scan with gitleaks in CI.

## Auth & authorization
- Supabase Auth with JWT; sessions in secure storage; refresh handled by SDK.
- Authorization lives in **RLS + Edge Functions**, never only in the UI.
- Admin role is set only in DB by a human; client cannot change `profiles.role` (policy + trigger).

## Payments
- Price is computed on the server in `create-order`. Client sends only `course_id` and optional `coupon_code`.
- Razorpay webhook: verify `X-Razorpay-Signature` (HMAC SHA256 with webhook secret) on the raw body; reject otherwise.
- Idempotency: unique `razorpay_payment_id` and `(user_id, course_id)`; replayed webhooks must not double-enroll.
- Never mark an order paid from the client callback. Client callback only triggers a refresh; webhook is the source of truth.

## Content protection
- Videos are private on Bunny (token authentication on). `get-playback-url` verifies enrollment or `is_free_preview`, returns URL valid ≤ 10 min.
- `bunny_video_id` is never selectable by clients (use `lessons_public` view).
- Android `FLAG_SECURE` on player screen (blocks screenshots/recording). Note: this deters casual copying, it is not full DRM.
- Max 2 active devices per account (`register-device`).
- PDF notes via short-lived signed URLs for enrolled users only.

## Data & privacy
- Collect minimum: name, email, optional photo. Provide Privacy Policy + Terms links in app and Play listing.
- Account deletion in-app (Play requirement) via `delete-account`.
- Do not log PII, tokens, or payment data. No `print` in release.
- Comply with India DPDP Act principles: consent, purpose limitation, deletion on request.

## Network & app hardening
- HTTPS only; `usesCleartextTraffic=false`.
- R8/ProGuard minify + Dart `--obfuscate`. Disable debug features in release.
- Rate limit Edge Functions (per user/IP), validate all inputs with a schema (zod), return generic error messages.
- Android: `allowBackup=false`; exported components minimal.

## Review checklist for every PR
- [ ] New table has RLS + policies + pgTAP test
- [ ] No secret in code or logs
- [ ] Server re-validates anything the client sends
- [ ] Error messages leak nothing internal
