# Ayurvedic Retreat Aggregation Platform

A full-stack aggregation and booking platform for Ayurvedic retreats — Booking.com purpose-built for Ayurveda, with genuine curation, no dark patterns, and Ayurveda-specific search (dosha, modality, certification).

Two-sided marketplace: seekers discover and book retreats; retreat centers manage their listings through a self-serve portal; the platform curates quality and takes a booking fee.

**Status:** Pre-build. Complete `INPUT-MARKETING-STRATEGY.md` and produce `style-guide.md` before writing any code.

---

## Mandatory Workflow — Do Not Skip Steps

Every session must follow this sequence. Never write code before the marketing strategy and style guide are locked.

### Step 1 — Marketing Strategy (one-time, before anything else)

Fill `INPUT-MARKETING-STRATEGY.md`. This defines the two-sided marketplace positioning, the primary audience bet, the one-line promise, and the copy tone. The design must follow from this document — not precede it.

Use the marketing panel to critique it:
- **April Dunford** — "Positioned against what? What are seekers switching from?" (positioned against generic booking sites and fragmented retreat center websites)
- **Mark Ritson** — Brand narrative: one-line promise + three proof points + emotional job-to-be-done
- **Byron Sharp** — Distinctiveness assets: what makes this brand saliently, memorably different over time

Marketing strategy is signed off when all three panels are satisfied.

### Step 2 — Design System (before any CSS)

Read `.claude/skills/frontend-design-SKILL.md`. Invoke the expert panel with the completed marketing strategy + Ayurvedic-retreat brief.

The panel:
- **James Fox** — Colour: cultural weight, emotional register, what the palette is not
- **Erik Spiekermann** — Typography: does this typeface do real work, or is it decoration?
- **Michael Bierut** — Brand identity: could a visitor identify this brand in 5 seconds without the logo?
- **Frank Chimero** — Web as medium: does this feel like a site, or a brochure pretending to be one?

The output is `style-guide.md` at the project root. No UI code is written before this file exists and the panel critique section is filled.

### Step 3 — Build (phases 1–9, see `PLAN.md`)

Follow the phase order in `PLAN.md`. Read `.claude/skills/ollama-supervisor-SKILL.md` before every coding session — route execution tasks to Ollama, reserve Claude for architecture, security, and synthesis.

---

## Skills Available

All skills live in `.claude/skills/`. Read the relevant SKILL.md to invoke.

| Skill file | When to use |
|---|---|
| `frontend-design-SKILL.md` | Phase 0 — design system, style guide, expert panel |
| `ollama-supervisor-SKILL.md` | Every coding session — cost reduction via Opus/Sonnet/Haiku + Ollama |
| `theme-factory/SKILL.md` | Phase 0 — pre-built palette starting points |
| `seo-landing-pages/SKILL.md` | Phase 3 — SEO-optimized landing pages with JSON-LD |
| `schema-markup-generator/SKILL.md` | Phase 3 — Generate JSON-LD for retreat detail, center, review pages |
| `webapp-testing/SKILL.md` | Phases 3–8 — browser QA with Playwright |
| `railway-deploy-SKILL.md` | Phase 9 — deployment readiness audit |

---

## Quality Gates (hard stops)

| Gate | Required before |
|---|---|
| `INPUT-MARKETING-STRATEGY.md` filled and panel-approved | Any design work |
| `style-guide.md` exists with Panel critique section | Any CSS or UI code |
| Auth flow tested end-to-end | Booking wizard |
| Booking creation works without payment | Stripe integration |
| Stripe webhook signature verification implemented | Stripe payment processing |
| Retreat detail pages have title, description, OG tags, JSON-LD | Phase 9 deployment |
| SEO: sitemap.xml and robots.txt routes exist | Phase 9 deployment |
| Keyboard navigation works; focus rings visible; ARIA labels on controls | Phase 7 QA |
| axe-core audit: zero violations on all pages | Phase 9 deployment |
| Center admin can only see their own data | Admin portal |
| All admin routes reject non-admin roles | Deployment |
| Health check `/health` returns 200 | Railway deployment |

---

## Tech Stack (when ready to build)

**Frontend** — `frontend/`
- React 19 + TypeScript, Vite 8
- Tailwind CSS v3 + CSS custom properties (RGB channel pattern — see `frontend-design-SKILL.md`)
- React Router v6
- framer-motion, lucide-react, react-hot-toast, react-day-picker
- @stripe/react-stripe-js

**Backend** — `backend/`
- Python FastAPI + Uvicorn
- SQLite WAL mode, raw sqlite3, no ORM
- bcrypt + PyJWT (email auth, role-based)
- Stripe Python SDK
- Reads `$PORT` from env (Railway requirement — never hardcode port)

---

## Reusable Code — Read Before Writing Anything New

Do not rewrite code that already exists in these projects:

| What to reuse | Source path |
|---|---|
| Typed fetch wrapper + 401 handling | `c:\Users\viren\source\property_broker_v2\frontend\src\lib\api.ts` |
| Auth hook (token lifecycle) | `c:\Users\viren\source\property_broker_v2\frontend\src\hooks\useAuth.ts` |
| Dark/light theme hook | `c:\Users\viren\source\property_broker_v2\frontend\src\hooks\useTheme.ts` |
| Tailwind config (RGB channel token pattern) | `c:\Users\viren\source\property_broker_v2\frontend\tailwind.config.js` |
| Vite proxy config for `/api` | `c:\Users\viren\source\property_broker_v2\frontend\vite.config.ts` |
| FastAPI app setup + CORS + router registration | `c:\Users\viren\source\property_broker_v2\backend\main.py` |
| SQLite `db_transaction()` context manager | `c:\Users\viren\source\property_broker_v2\backend\database.py` |
| JWT issue/verify pattern | `c:\Users\viren\source\property_broker_v2\backend\auth.py` |
| Dockerfile + railway.toml (monorepo) | `c:\Users\viren\source\property_broker_v2\` |
| Multi-step wizard shell | `C:\Users\viren\source\identitypurpose\frontend\src\components\WorkshopShell.tsx` |
| Step layout wrapper | `C:\Users\viren\source\identitypurpose\frontend\src\stages\StageTemplate.tsx` |
| Option pill + textarea question block | `C:\Users\viren\source\identitypurpose\frontend\src\components\QuestionBlock.tsx` |
| VITE_API_URL env pattern | `C:\Users\viren\source\identitypurpose\frontend\src\api\env.ts` |

---

## Design Aesthetic Starting Point

**Proposed north star** (to be confirmed by expert panel in style guide):
"The trusted guide — warm authority, not spa-brochure gloss. The confidence of expertise, the warmth of a personal recommendation."

**Theme-factory reference:** Forest Canopy (`#2d4a2b` deep forest, `#faf9f6` ivory) + Botanical Garden (marigold `#f9a620`, terracotta `#b7472a`) — diverge toward saffron-gold CTA and parchment background.

The expert panel has final say. The proposed north star is a hypothesis, not a decision.

---

## Database Overview (see `PLAN.md` for full schema)

Six core tables: `retreat_centers`, `retreats`, `retreat_dates`, `users`, `bookings`, `consultations`, `reviews`.

User roles: `seeker` | `center_admin` | `platform_admin`

Booking statuses: `pending_payment` | `confirmed` | `cancelled` | `completed`

Consultation statuses: `new` | `contacted` | `matched` | `booked`

---

## Environment Variables

Never commit these. Set in Railway dashboard.

| Variable | Service | Notes |
|---|---|---|
| `JWT_SECRET` | Backend | Long random string (min 32 chars) |
| `STRIPE_SECRET_KEY` | Backend | From Stripe dashboard (sk_test_...) |
| `STRIPE_WEBHOOK_SECRET` | Backend | From Stripe webhook config (whsec_...) |
| `VITE_API_URL` | Frontend | Set before first deploy — baked at build time |
| `VITE_STRIPE_PK` | Frontend | Stripe publishable key (pk_test_...) |
| `ALLOWED_ORIGINS` | Backend | Exact frontend URL with `https://` |
| `RESEND_API_KEY` | Backend | Resend email service API key (re_...) |
| `EMAIL_FROM` | Backend | Sender email address (e.g., bookings@yourdomain.com) |
| `PORT` | Backend | Railway sets automatically; local dev defaults to 8000 |
| `DATABASE_URL` | Backend | Leave blank for SQLite; set for Railway Postgres |

---

## Copy & Tone Rules

- Second-person voice: "you", "your"
- No: "world-class", "innovative", "holistic journey", "transformative", "premium", "best-in-class"
- CTAs are action-specific: never "Submit", "Learn More", "Click here"
- Headlines lead with outcome or benefit, not feature
- Respect: no fake scarcity, no hidden fees, pricing shown upfront

## WCAG

Body text ≥ 4.5:1. Large display text ≥ 3:1. Test muted text on both light and dark backgrounds.
