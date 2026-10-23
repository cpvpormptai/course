# 01 — Product Requirements Document

## Vision
A simple, fast Android app where learners discover, buy, and watch video courses, and instructors/admins publish them.

## Assumptions (change here if wrong)
- Single publisher first (your own courses); multi-instructor marketplace is a later phase.
- Market: India first. Currency INR, payments via Razorpay (UPI, cards, netbanking). Languages: English + Hindi UI.
- Content: recorded video (HLS) + PDF notes. Live classes are out of scope for v1.
- Google Play rule: digital content sold for use inside the app may need Google Play Billing. Verify current Play policy before launch; the `PaymentGateway` interface exists so we can swap or add Play Billing.

## Personas
- **Learner:** wants to find a course, trust it, pay in under a minute, resume where they stopped.
- **Admin/Instructor:** wants to upload videos, set price, see sales and students.

## MVP scope (v1)
| ID | Feature | Acceptance criteria |
|----|---------|--------------------|
| F1 | Auth | Google sign-in and email OTP. Session persists. Sign-out works. |
| F2 | Catalog | Home with featured + categories; search by title; course detail with syllabus, price, instructor, rating. |
| F3 | Free preview | Lessons marked preview play without login/purchase. |
| F4 | Purchase | Buy course via Razorpay; on success enrollment appears within 10 s; failed payment shows retry; coupon code supported. |
| F5 | My Courses | List of enrolled courses with progress %. |
| F6 | Player | HLS playback, speed control, resume position, quality auto, screenshot blocked (FLAG_SECURE). |
| F7 | Progress | Lesson marked complete at 90% watched; course progress % updates. |
| F8 | Reviews | Enrolled users can rate 1–5 and write a review once per course. |
| F9 | Notifications | Push for purchase confirmation and new lesson added. |
| F10 | Profile | Name, photo, language switch (EN/HI), purchase history, sign-out, delete account. |
| F12 | 3D animation | Play-cube animation on Splash, Home hero, Payment success, Course completed (see 08). Respects reduce-motion, low-end fallback, Profile on/off switch. |
| F11 | Admin (web, P6) | Create/edit course, sections, lessons, upload video, set price/coupons, view orders. |

## Out of scope for v1
Live classes, in-app chat, subscriptions, iOS, multi-instructor payouts, offline downloads, quizzes.

## Key flows
1. **Buy:** Course detail → Buy → server creates order → Razorpay sheet → webhook confirms → enrollment created → "Start learning".
2. **Watch:** My Courses → lesson → app asks server for playback URL → server checks enrollment → signed short-lived HLS URL → play.

## Non-functional requirements
- Cold start < 3 s on a mid-range phone; list screens render from cache first.
- Works on Android 8+ (API 26+).
- Handles poor networks: retry, timeouts, clear offline message.
- Accessibility: min touch target 48dp, contrast AA, scalable text.

## Success metrics
Install → signup conversion, course page → purchase conversion, D7 retention, lesson completion rate, refund rate, crash-free sessions > 99%.

## Open questions (human must decide)
Course niche/name, pricing model, refund policy, GST invoicing, whether Play Billing is required for your case.
