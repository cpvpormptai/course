# 03 — Database (Supabase Postgres)

## Rules
- All tables in `public`, RLS **enabled on every table**. No exceptions.
- Money = integer paise (`price_paise`). Currency fixed `INR` in v1.
- IDs = `uuid` (`gen_random_uuid()`), timestamps = `timestamptz` default `now()`.
- Every schema change is a migration file in `supabase/migrations/`. Never edit the DB by hand.
- Writes to `orders`, `payments`, `enrollments` happen only via Edge Functions (service role). Clients can never insert them.

## Schema
```sql
create type user_role as enum ('learner','admin');
create type order_status as enum ('created','paid','failed','refunded');

create table profiles (
  id uuid primary key references auth.users on delete cascade,
  full_name text, avatar_url text, language text default 'en',
  role user_role not null default 'learner',
  created_at timestamptz not null default now()
);

create table categories (
  id uuid primary key default gen_random_uuid(),
  name_en text not null, name_hi text, sort_order int default 0
);

create table courses (
  id uuid primary key default gen_random_uuid(),
  category_id uuid references categories,
  title text not null, subtitle text, description text,
  thumbnail_url text, language text default 'hi',
  price_paise int not null check (price_paise >= 0),
  mrp_paise int check (mrp_paise >= price_paise),
  is_published boolean not null default false,
  rating_avg numeric(2,1) default 0, rating_count int default 0,
  created_at timestamptz not null default now()
);

create table sections (
  id uuid primary key default gen_random_uuid(),
  course_id uuid not null references courses on delete cascade,
  title text not null, sort_order int not null default 0
);

create table lessons (
  id uuid primary key default gen_random_uuid(),
  section_id uuid not null references sections on delete cascade,
  course_id uuid not null references courses on delete cascade,
  title text not null, duration_sec int,
  bunny_video_id text,            -- never exposed to clients
  notes_path text,                -- storage path for PDF
  is_free_preview boolean not null default false,
  sort_order int not null default 0
);

create table coupons (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,
  percent_off int check (percent_off between 1 and 100),
  flat_off_paise int check (flat_off_paise > 0),
  max_uses int, used_count int default 0,
  expires_at timestamptz, is_active boolean default true
);

create table orders (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles,
  course_id uuid not null references courses,
  coupon_id uuid references coupons,
  amount_paise int not null,
  status order_status not null default 'created',
  razorpay_order_id text unique,
  created_at timestamptz not null default now()
);

create table payments (
  id uuid primary key default gen_random_uuid(),
  order_id uuid not null references orders,
  razorpay_payment_id text unique not null,
  raw_event jsonb, created_at timestamptz not null default now()
);

create table enrollments (
  user_id uuid not null references profiles,
  course_id uuid not null references courses,
  order_id uuid references orders,
  created_at timestamptz not null default now(),
  primary key (user_id, course_id)
);

create table lesson_progress (
  user_id uuid not null references profiles,
  lesson_id uuid not null references lessons on delete cascade,
  position_sec int not null default 0,
  completed boolean not null default false,
  updated_at timestamptz not null default now(),
  primary key (user_id, lesson_id)
);

create table reviews (
  user_id uuid not null references profiles,
  course_id uuid not null references courses,
  rating int not null check (rating between 1 and 5),
  body text, created_at timestamptz not null default now(),
  primary key (user_id, course_id)
);

create table devices (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles,
  device_id text not null, fcm_token text,
  last_seen timestamptz default now(),
  unique (user_id, device_id)
);

create index on courses (is_published, category_id);
create index on lessons (course_id, sort_order);
create index on enrollments (user_id);
```

## RLS policies (summary — implement each as SQL)
| Table | Read | Write |
|-------|------|-------|
| profiles | own row (admin: all) | own row, except `role` |
| categories | everyone | admin |
| courses, sections | everyone if `is_published` | admin |
| lessons | everyone, **via view `lessons_public` without `bunny_video_id`** | admin |
| coupons | nobody (server validates) | admin |
| orders, payments | own rows | none (service role only) |
| enrollments | own rows | none (service role only) |
| lesson_progress | own rows | own rows, only for enrolled courses |
| reviews | everyone | own row, only if enrolled |
| devices | own rows | via Edge Function |

Helper: `create function is_admin() returns boolean ... security definer` and `is_enrolled(course uuid)`.
Revoke direct select on `lessons.bunny_video_id` from `anon, authenticated`; expose `lessons_public` view instead.

## Triggers
- On `auth.users` insert → create `profiles` row.
- On `reviews` insert/update/delete → recompute `courses.rating_avg/count`.
- On `coupons` use → increment `used_count` inside `create-order`.

## Storage buckets
`avatars` (public read, own write), `thumbnails` (public read, admin write), `notes` (private; signed URLs only for enrolled users).

## Seed
`supabase/seed.sql` must create: 3 categories, 3 courses, sections/lessons (one free preview each), 1 coupon, 1 admin user (dev only).
