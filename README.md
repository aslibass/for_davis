# Ayurvedic Retreat Aggregation Platform

A full-stack **Booking.com for Ayurveda** — a discovery-first marketplace where seekers find, compare, and book authentic Ayurvedic retreats. Multiple retreat centers list their programs. The platform curates quality and takes a booking fee.

**Status:** Pre-build. All documentation locked. Ready to begin Phase 0 (marketing strategy + design system).

---

## What You're Looking At

This repository is a **production-ready project scaffold** — not a partially-built app. It contains:

- ✅ **Architecture decisions** (locked in `decisions.md`)
- ✅ **Implementation plan** (`PLAN.md`) — 9 phases from design system to deployment
- ✅ **Pre-flight checklists** (quality gates in `CLAUDE.md`)
- ✅ **Reusable skills** (`.claude/skills/` — design, deployment, testing, cost optimization)
- ✅ **Configuration templates** (`.env.example`, `style-guide.md` template)
- ❌ **Zero application code** (starts in Phase 1, after design system is locked)

All critical decisions have been made so you don't have to re-decide them. The pre-build took 8 hours and eliminated 8 common pitfalls (SEO, email, accessibility, Stripe architecture, center onboarding, etc.). Read `CLAUDE.md` to understand what's here and what's locked.

---

## Quick Start

### Prerequisites
- Node.js 18+
- Python 3.10+
- Railway account (for deployment)
- Stripe account (for payments)
- Resend account (for email)
- Git

### Step 1 — Fill the Marketing Strategy

```bash
# Open this file and answer every section
# The expert panel (April Dunford, Mark Ritson, Byron Sharp) will critique your answers
INPUT-MARKETING-STRATEGY.md
```

The design system cannot be created until the marketing strategy is locked.

### Step 2 — Generate the Design System

```bash
# Use the built-in frontend-design skill
# It will invoke the expert panel (Fox, Spiekermann, Bierut, Chimero) to critique
# and produce style-guide.md with your colour tokens, fonts, and component rules

/frontend-design
```

See `.claude/skills/frontend-design-SKILL.md` for details.

### Step 3 — Begin the Build

```bash
# Follow the phase order in PLAN.md
# Phase 1: scaffold (Vite + FastAPI + SQLite)
# Phase 2: auth (email + JWT)
# Phase 3: public pages (landing, search, detail)
# ... through Phase 9: deployment

# Use the model-tier routing strategy:
/model opus      # for planning and security decisions
/model sonnet    # for complex multi-file features
/model haiku     # for single-file boilerplate
```

See `.claude/skills/ollama-supervisor-SKILL.md` for the detailed routing strategy.

---

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | **Read this first.** Mandatory workflow, tech stack, reusable code, quality gates |
| `PLAN.md` | 9 implementation phases with routes, API endpoints, database schema, verification steps |
| `INPUT-MARKETING-STRATEGY.md` | Marketing strategy template (Dunford/Ritson/Sharp panel critique) |
| `style-guide.md` | Design system output — colour tokens, typography, motion (locked by expert panel) |
| `decisions.md` | Append-only log of architectural decisions (north star aesthetic, Stripe direct charge, Resend for email, etc.) |
| `.env.example` | All environment variables documented with comments |
| `.claude/skills/` | Reusable skills: frontend-design, railway-deploy, ollama-supervisor, theme-factory, webapp-testing |

---

## Architecture Overview

### Two-Sided Marketplace

**Supply side:** Retreat centers  
- Self-service registration (`/list-your-retreat-center`)
- Application form submission (`POST /api/center-admin/apply`)
- Platform admin approval (Phase 8)
- Center admin portal (manage retreats, dates, bookings, reviews)

**Demand side:** Seekers  
- Discovery (browse, search, filter by dosha/modality/price/duration)
- Dosha quiz → personalized retreat recommendations
- Booking wizard (3-step: guests, health intake, payment)
- Consultation concierge ("help me find my retreat")

### Tech Stack

**Frontend**
- React 19 + TypeScript
- Vite 8 (build tool)
- Tailwind CSS v3 + CSS custom properties (design token pattern)
- React Router v6 (page routing)
- framer-motion (animations)

**Backend**
- Python FastAPI + Uvicorn
- SQLite (local) / PostgreSQL (production)
- bcrypt + PyJWT (email auth + roles)
- Stripe Python SDK (payments)
- Resend SDK (transactional email)

**Deployment**
- Railway (monorepo: two services, one for backend, one for frontend)
- Docker (containerization)

### Database Schema

Six core tables: `retreat_centers`, `retreats`, `retreat_dates`, `users`, `bookings`, `consultations`, `reviews`

User roles: `seeker` | `center_admin` | `platform_admin`

See `PLAN.md` for full schema.

---

## Design System (Not Yet Locked)

The aesthetic direction has been **proposed** but not yet locked by the expert panel:

> **Proposed North Star:** "Contemplative ashram — aged parchment against deep forest, meditative serif type, the stillness of a mountain retreat rather than the brightness of a spa brochure."

**Candidate palette (Forest Canopy diverged):**
- Background primary: warm parchment (ivory)
- Background feature: deep forest green (one dark section per page)
- Accent CTA: saffron gold (used once per section)
- Typography: meditative serif (Cormorant Garamond or Playfair Display) + refined body sans

This will be confirmed or refined when you run `/frontend-design` skill with the marketing strategy.

---

## Critical Architecture Decisions (Locked)

### 1. Payment (Stripe Direct Charge MVP)

**Decision:** Option A — direct charge, no Stripe Connect in MVP

- Seeker pays full price to platform via Stripe Checkout
- Platform pays centers separately (manual transfer or ACH in v2)
- Defer Stripe Connect to v2 (simpler to test, deploy, and scale)
- **Mandatory:** Webhook signature verification on every webhook call
- **Mandatory:** Background job cancels `pending_payment` bookings after 2 hours

See `decisions.md` for full rationale.

### 2. Email (Resend)

**Decision:** Resend for transactional email

- API-key only (no SMTP server)
- Generous free tier (100 emails/day)
- Excellent deliverability
- Simple Python SDK integration

Emails sent at:
- Booking confirmed (seeker + center)
- Consultation submitted (acknowledgement)
- Center approved (with portal login link)

### 3. Center Onboarding (Self-Service + Approval)

**Decision:** Centers self-register, then wait for platform admin approval

- Flow: register → submit application form → admin approves → approval email → access portal
- Lowers friction for centers (no support burden)
- Approval gate ensures quality and curation (trust is our positioning advantage)

### 4. Stripe Webhook Verification (Critical Security)

**Decision:** Mandatory signature verification on all webhook calls

This is non-negotiable. Without it, any attacker can forge webhook calls and fake payment confirmations.

See Phase 5 in `PLAN.md` for implementation details.

---

## Quality Gates (Must Pass Before Deployment)

| Gate | When |
|------|------|
| Stripe webhook signature verification implemented | Before Phase 5 payment processing |
| SEO: meta/OG/JSON-LD on all public pages | Before Phase 9 deployment |
| Accessibility: keyboard nav, ARIA labels, axe-core clean | Before Phase 9 deployment |
| Admin routes reject non-admin roles | Before Phase 9 deployment |
| Health check `/health` returns 200 | Before Railway deployment |

---

## Project Reuse — Copy Skills to Global

All skills are designed to be reusable in other projects. Copy them:

```bash
cp .claude/skills/*.md C:\Users\viren\.claude\skills\
cp -r .claude/skills/theme-factory C:\Users\viren\.claude\skills\
cp -r .claude/skills/webapp-testing C:\Users\viren\.claude\skills\
cp -r .claude/skills/seo-landing-pages C:\Users\viren\.claude\skills\
cp -r .claude/skills/schema-markup-generator C:\Users\viren\.claude\skills\
```

All 7 skills are already in the global directory and ready for other projects.

### Skills Included

**Custom skills for this project:**
1. **frontend-design-SKILL.md** — Design system creation with expert panel (Fox/Spiekermann/Bierut/Chimero)
2. **ollama-supervisor-SKILL.md** — Two-tier cost optimization (Opus/Sonnet/Haiku routing + Ollama fallback)
3. **railway-deploy-SKILL.md** — Railway deployment config and audit
4. **theme-factory/** — 10 pre-built design theme palettes (Forest Canopy, Botanical Garden, Desert Rose, etc.)
5. **webapp-testing/** — Playwright browser testing with managed server lifecycle

**Community skills (fetched from Anthropic + open-source repos):**
6. **seo-landing-pages/** — SEO-optimized landing pages with JSON-LD structured data (anthropics/skills)
7. **schema-markup-generator/** — JSON-LD generation for retreats, centers, reviews (aaron-he-zhu/seo-geo-claude-skills)

---

## How to Use This Repository

### For This Project
1. **Read `CLAUDE.md`** — mandatory workflow and quality gates
2. **Read `PLAN.md`** — architecture, phases, routes, API, database schema
3. **Fill `INPUT-MARKETING-STRATEGY.md`** — answer Dunford/Ritson/Sharp questions
4. **Run `/frontend-design` skill** — generate `style-guide.md` with expert panel
5. **Build Phases 1–9** using the model-tier routing in `ollama-supervisor-SKILL.md`

### For Other Projects
1. Copy `ollama-supervisor-SKILL.md` to your global skills
2. Copy `frontend-design-SKILL.md` to your global skills (expert panel included)
3. Copy `railway-deploy-SKILL.md` if deploying to Railway
4. Copy `theme-factory/` if you want design theme starting points
5. Copy `webapp-testing/` if you want browser automation

---

## Environment Variables

Never commit `.env` files. See `.env.example` for all variables.

Local development:
```bash
cp .env.example .env
# Fill in your values (for local dev, you can use test keys)
```

Production (Railway):
```
# Set in Railway dashboard, never in code:
JWT_SECRET=<long-random-string>
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
RESEND_API_KEY=re_...
EMAIL_FROM=bookings@yourdomain.com
VITE_API_URL=https://api.yourdomain.com
VITE_STRIPE_PK=pk_live_...
ALLOWED_ORIGINS=https://yourdomain.com
DATABASE_URL=postgresql://user:pass@host/db
```

---

## "Better Than Booking.com" Commitments

These are non-negotiable product rules built into every phase:

1. **No dark patterns** — no fake scarcity timers, no hidden fees, price shown upfront
2. **Ayurveda-first filtering** — filter by dosha, modality, practitioner certification (not just stars)
3. **Genuine curation** — every listing approved; center certifications verified before listing
4. **Health intake in booking flow** — structured form, sent to center, not an afterthought email
5. **Concierge always available** — "help me find my retreat" link on every page
6. **Review authenticity** — only guests with confirmed completed bookings can review

---

## Getting Help

- **Mandatory workflow?** → Read `CLAUDE.md`
- **Build phases?** → Read `PLAN.md`
- **Design system?** → See `.claude/skills/frontend-design-SKILL.md`
- **Cost optimization?** → See `.claude/skills/ollama-supervisor-SKILL.md`
- **Deployment?** → See `.claude/skills/railway-deploy-SKILL.md`
- **Architectural decisions?** → Read `decisions.md`

---

## Next Steps

```bash
# Step 1: Understand the project
cat CLAUDE.md

# Step 2: Review the implementation plan
cat PLAN.md

# Step 3: Start the mandatory workflow
# Open INPUT-MARKETING-STRATEGY.md and fill every section

# Step 4: Generate design system
# Run /frontend-design (inside Claude Code)

# Step 5: Build Phase 1 scaffold
# Follow PLAN.md Phase 1 step by step
```

---

**Built with:** Claude Opus 4.8 (architecture), Sonnet 4.6 (planning), Haiku 4.5 (execution)  
**Last updated:** 2026-06-04  
**License:** Internal (Viren)
