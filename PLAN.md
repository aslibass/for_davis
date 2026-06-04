# Ayurvedic Retreat Aggregation Platform — Implementation Plan

## Context

Building a full-stack **aggregation and booking platform** for Ayurvedic retreats — think Booking.com but purpose-built for Ayurveda, with far better UX, genuine curation, and no dark patterns. Multiple retreat centers list their programs; seekers discover, compare, and book. The platform sits in the middle and takes a booking fee.

Key differences from a single-retreat booking site:
- Multi-vendor: many retreat centers, many programs
- Discovery-first UX: search → filter → compare → book
- Two-sided marketplace: seekers + retreat center admins
- Platform curation: listings are approved before going live
- Ayurveda-specific: dosha types, treatment modalities, practitioner credentials matter

Target audience (from marketing strategy): serious wellness seekers who want authentic Ayurvedic experiences but currently have no trustworthy, curated source — they're piecing together information from fragmented websites.

---

## Mandatory Workflow — Follow This Order

1. Fill `INPUT-MARKETING-STRATEGY.md` (two-sided marketplace positioning)
2. Invoke `/frontend-design` skill — produce `style-guide.md` before any UI code
3. Expert panel (Fox / Spiekermann / Bierut / Chimero) signs off on style-guide
4. Build by phase, Ollama executes, Claude supervises

---

## Skills to Use (in order)

| Skill | When |
|---|---|
| `frontend-design` | Phase 0 — before any CSS. Produces `style-guide.md`. |
| `ollama-supervisor` | Every coding session — route execution to Ollama. |
| `theme-factory` | Phase 0 — starting palette reference before diverging. |
| `webapp-testing` | Phases 3–7 — QA after each major page build. |
| `railway-deploy` | Phase 8 — deployment readiness audit. |

---

## Tech Stack

| Layer | Choice | Source |
|---|---|---|
| Frontend | React 19 + TypeScript + Vite 8 | property_broker_v2 |
| Styling | Tailwind CSS v3 + CSS custom properties | property_broker_v2 |
| Routing | React Router v6 | new (required) |
| Animations | framer-motion | property_broker_v2 |
| Icons | lucide-react | property_broker_v2 |
| Toasts | react-hot-toast | property_broker_v2 |
| Date picker | react-day-picker | new |
| Payments | @stripe/react-stripe-js | property_broker_v2 |
| Backend | Python FastAPI + Uvicorn | property_broker_v2 |
| Database | SQLite (WAL mode, raw sqlite3, no ORM) | property_broker_v2 |
| Auth | Email + bcrypt + JWT | adapted from property_broker_v2 |
| Payments | Stripe Checkout (redirect) | property_broker_v2 |
| Deployment | Railway monorepo (two services) | property_broker_v2 |

---

## Database Schema

```sql
-- Retreat centers (the vendors)
retreat_centers (
  id, slug, name, tagline, description,
  city, country, address,
  contact_email, website_url,
  image_url, gallery_json,         -- images
  certifications_json,             -- e.g. NABH, Kerala Ayurveda Council
  is_approved, approved_at,
  owner_user_id,                   -- center admin
  created_at
)

-- Retreat programs offered by a center
retreats (
  id, center_id, slug, title, tagline, description,
  dosha_focus,                     -- vata | pitta | kapha | tridosha | general
  modality,                        -- panchakarma | rasayana | yoga_ayurveda | detox | custom
  duration_days, price_usd_from,
  max_guests, min_age,
  what_is_included_json,
  what_is_not_included_json,
  image_url, gallery_json,
  is_active, created_at
)

-- Available date slots for a retreat
retreat_dates (
  id, retreat_id, start_date, end_date,
  available_spots, price_usd,
  is_open_for_booking
)

-- Platform users (seekers + center admins + platform admins)
users (
  id, email, password_hash, name, phone,
  role,                            -- seeker | center_admin | platform_admin
  created_at
)

-- Bookings
bookings (
  id, user_id, retreat_date_id,
  status,                          -- pending_payment | confirmed | cancelled | completed
  stripe_session_id,
  guest_count, total_usd,
  health_intake_json,              -- dosha self-assessment, conditions, goals, dietary needs
  created_at
)

-- Consultation requests (help me find the right retreat)
consultations (
  id, name, email, phone,
  budget_usd, preferred_duration_days,
  health_goals, dosha_self_assessment,
  preferred_start_month, is_flexible,
  status,                          -- new | contacted | matched | booked
  matched_retreat_id,
  created_at
)

-- Reviews (post-stay)
reviews (
  id, user_id, retreat_id, booking_id,
  rating,                          -- 1-5
  body, response_by_center,
  is_approved, created_at
)
```

---

## Pages & Routes

```
/                          Discovery home (search hero, featured retreats, dosha quiz CTA)
/retreats                  Browse all retreats (search + filters)
/retreats/:slug            Retreat detail (photos, description, dates, reviews, book)
/centers/:slug             Retreat center profile (about, all their programs)
/compare                   Side-by-side retreat comparison (up to 3)
/quiz                      Dosha quiz → personalised retreat recommendations
/consultation              "Help me find my retreat" concierge request

/auth/login                Email sign-in
/auth/register             Email sign-up

/dashboard                 Seeker: booking history, upcoming retreats, saved retreats
/booking/:retreatDateId    Booking wizard (auth-gated)
  ?step=1                  → Guest count + special requests
  ?step=2                  → Health intake (dosha, conditions, goals, diet)
  ?step=3                  → Review & pay (Stripe redirect)
/booking/success           Post-payment confirmation

/center-admin              Center dashboard (my listings, bookings, analytics)
/center-admin/retreats     Manage retreat programs
/center-admin/dates        Manage available dates
/center-admin/bookings     View bookings for my center
/center-admin/reviews      Respond to reviews

/admin                     Platform admin home
/admin/centers             Approve / reject center applications
/admin/retreats            Approve / moderate retreat listings
/admin/bookings            All bookings across all centers
/admin/consultations       Consultation requests queue
/admin/reviews             Moderate reviews
```

---

## Backend API

```
POST /api/auth/register
POST /api/auth/login
GET  /api/auth/me

GET  /api/retreats                     search + filter (location, duration, dosha, price, dates)
GET  /api/retreats/:slug
GET  /api/retreats/:slug/dates
GET  /api/retreats/:slug/reviews

GET  /api/centers/:slug
GET  /api/centers/:slug/retreats

GET  /api/compare?ids=a,b,c            side-by-side data for up to 3 retreats

POST /api/quiz/result                  submit dosha quiz → return matched retreat slugs

POST /api/consultations
POST /api/bookings                     → create pending booking + Stripe checkout session
GET  /api/bookings                     user's own bookings
GET  /api/bookings/:id
POST /api/reviews                      post-stay review (requires completed booking)

POST /api/payments/webhook             Stripe → confirm booking

# Center admin (requires role=center_admin + owns the center)
GET  /api/center-admin/center
PATCH /api/center-admin/center
POST /api/center-admin/retreats
PATCH /api/center-admin/retreats/:id
POST /api/center-admin/dates
PATCH /api/center-admin/dates/:id
GET  /api/center-admin/bookings
POST /api/center-admin/reviews/:id/respond

# Platform admin
GET  /api/admin/centers                (all, with pending filter)
PATCH /api/admin/centers/:id/approve
GET  /api/admin/retreats
PATCH /api/admin/retreats/:id/approve
GET  /api/admin/bookings
GET  /api/admin/consultations
PATCH /api/admin/consultations/:id
```

---

## Reusable Code (do not rewrite)

### From `c:\Users\viren\source\property_broker_v2`
- `frontend/src/lib/api.ts` — typed fetch wrapper; adapt endpoint names
- `frontend/src/hooks/useAuth.ts` — token lifecycle; adapt for email JWT
- `frontend/src/hooks/useTheme.ts` — dark/light toggle
- `frontend/vite.config.ts` — Vite proxy config for `/api`
- `frontend/tailwind.config.js` — RGB channel token pattern; replace colour values from style-guide
- `backend/auth.py` — JWT issue/verify; swap Google PKCE for email/password + role support
- `backend/database.py` — `db_transaction()` context manager; use as-is
- `backend/main.py` — FastAPI app setup; use router registration pattern
- `Dockerfile` + `railway.toml` — copy and update for monorepo

### From `C:\Users\viren\source\identitypurpose\frontend`
- `src/components/WorkshopShell.tsx` → `BookingWizard.tsx` (multi-step shell)
- `src/stages/StageTemplate.tsx` → `StepTemplate.tsx` (step layout wrapper)
- `src/components/QuestionBlock.tsx` → health intake + dosha quiz question component
- `src/api/client.ts` — `get<T>` / `post<T>` typed fetch pattern
- `src/api/env.ts` — `VITE_API_URL` pattern

---

## Build Order

### Phase 0 — Design System (mandatory first)
1. Fill `INPUT-MARKETING-STRATEGY.md`
2. Run `/frontend-design` skill with marketing strategy + Ayurvedic brief
3. Expert panel critique (Fox / Spiekermann / Bierut / Chimero)
4. Produce `style-guide.md`
5. No code before `style-guide.md` is locked

### Phase 1 — Scaffold
1. `npm create vite@latest frontend -- --template react-ts`
2. Install all deps (see tech stack)
3. Port Tailwind config + CSS variable architecture from property_broker_v2
4. Apply style-guide colour tokens and font imports
5. Create FastAPI backend structure + SQLite schema init

### Phase 2 — Auth + Roles
1. `backend/routers/auth.py` — register/login/me with role field
2. `frontend/src/hooks/useAuth.ts` — email JWT, role-aware
3. Login + Register pages (form pattern from identitypurpose JoinScreen)
4. Route guard components: `RequireAuth`, `RequireRole`

### Phase 3 — Discovery UX (public pages)
1. `Landing.tsx` — search hero, dosha quiz CTA, featured retreats, "why us" strip
2. `RetreatSearch.tsx` — results grid + filter panel (location, duration, dosha, price range, dates)
3. `RetreatDetail.tsx` — photo gallery, description, dates picker, reviews, "Book" + "Save" CTAs
4. `CenterProfile.tsx` — center about, certifications, all their retreats
5. `Compare.tsx` — side-by-side up to 3 retreats (add from detail page)

### Phase 4 — Dosha Quiz + Consultation
1. `DoshaQuiz.tsx` — 8-10 question quiz → scored → retreat type recommendation
2. `Consultation.tsx` — concierge form: budget, duration, health goals, dosha, dates

### Phase 5 — Booking Wizard
1. `BookingWizard.tsx` — shell from WorkshopShell; step in URL param
2. Step 1: guest count, special requests, accessibility needs
3. Step 2: health intake (dosha self-assessment, medical conditions, dietary, goals)
4. Step 3: review summary + Stripe Checkout redirect
5. `BookingSuccess.tsx` — confirmation with booking details
6. Stripe webhook → update booking `status = confirmed`

### Phase 6 — Seeker Dashboard
1. `Dashboard.tsx` — upcoming retreats, past bookings, saved retreats, profile
2. Booking card: retreat name + center, dates, status badge, review CTA (post-stay)
3. Review submission flow

### Phase 7 — Center Admin Portal
1. Center dashboard with booking overview
2. Retreat CRUD (create/edit/archive programs)
3. Date slot management
4. Booking list view (read-only, with guest health intake)
5. Review response

### Phase 8 — Platform Admin
1. Center application approval queue
2. Retreat listing moderation
3. All bookings table (exportable)
4. Consultation queue with assignment/status

### Phase 9 — Deployment
1. Invoke `/railway-deploy` skill
2. Two Railway services: `backend/` and `frontend/`
3. `railway.toml` in each subdirectory
4. Environment variables: `JWT_SECRET`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `VITE_API_URL`, `ALLOWED_ORIGINS`
5. Health check endpoint `/health` on backend

---

## Verification Plan

- Phase 3 → `/run` to walk landing, search, and detail pages in browser
- Phase 5 → `/verify` full booking flow with Stripe test card (4242 4242 4242 4242)
- Phase 7 → `/code-review` security focus — center admin can only see their own data
- Phase 8 → `/code-review` platform admin only — no seeker can access admin routes
- Phase 9 → `/verify` Railway deployment health check

---

## "Better Than Booking.com" Commitments

These are non-negotiable product differences:

1. **No dark patterns** — no fake scarcity timers, no hidden fees, price shown upfront
2. **Ayurveda-first filtering** — filter by dosha, modality, practitioner certification — not just stars and price
3. **Genuine curation** — every listing approved; center certifications verified before listing
4. **Health intake in the booking flow** — not an afterthought email; structured and sent to the center
5. **Concierge option on every page** — always one click from "help me choose"
6. **Review authenticity** — only guests with confirmed completed bookings can review

---

## Design Aesthetic (to be confirmed by expert panel)

Proposed north star: **"The trusted guide — warm authority, not spa-brochure gloss. The confidence of expertise, the warmth of a personal recommendation."**

Candidate palette starting point: Forest Canopy + Botanical Garden diverged.
- Background primary: parchment/ivory (warm, not clinical white)
- Background feature: deep forest (used for one section — philosophy or trust statement)
- Accent CTA: saffron gold (the warm, specific Ayurvedic accent colour — used sparingly)
- Typography: meditative display serif (Cormorant Garamond or Playfair Display) + clean body sans

Expert panel has final say. No CSS before `style-guide.md` is signed off.
