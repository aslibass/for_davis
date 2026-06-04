# Design Decisions — Ayurvedic Retreat Aggregation Platform

Append-only log of major design and architectural decisions. All reversible decisions are recorded here.

**Entry schema:** Date | Topic | Decision | Rationale | Alternatives considered | Panel sign-off (if applicable)

---

## 2026-06-04 — Aesthetic North Star Direction

**Topic:** Design system aesthetic direction (Phase 0 pre-approval)

**Decision:** Contemplative ashram aesthetic — aged parchment background against deep forest green, meditative serif typography (Cormorant Garamond or Playfair Display), saffron-gold CTA accents. The stillness of a mountain retreat, not the brightness of a spa brochure.

**Rationale:** Ayurvedic wellness is about quiet authority and depth, not luxury marketing. The palette should read as "trustworthy expertise" — warm, grounded, intentional — to seekers evaluating serious health decisions. Parchment + forest + serif is distinctive from generic wellness sites and booking platforms.

**Alternatives considered:**
- Botanical/vibrant (marigold + terracotta): too energetic, reads "marketplace" not "authority"
- Minimalist/tech (cool darks + sans): contradicts the meditative, analog quality of Ayurveda

**Panel sign-off:** Pending. Expert panel critique (Fox/Spiekermann/Bierut/Chimero) will lock this in Phase 0 style-guide generation.

---

## 2026-06-04 — Stripe Payment Architecture (MVP)

**Topic:** Payment collection and center commission model

**Decision:** Option A (Direct charge, no Stripe Connect in MVP). Seeker pays full retreat price to platform via Stripe Checkout. Platform pays centers separately (manual transfer or future ACH integration). Defer Stripe Connect marketplace integration to v2.

**Rationale:** Direct charge is simpler to implement, test, and deploy quickly. Stripe Connect requires centers to onboard to Stripe (additional friction, chargeback liability transferred to them). For MVP with unknown volume, simpler is safer. Once we have 50+ centers and predictable volume, Connect's automated settlement justifies the complexity.

**Alternatives considered:**
- Stripe Connect: correct for mature marketplace, too complex for MVP testing
- PayPal Adaptive Payments: legacy, lower adoption in international wellness market
- Manual payment (offline bank transfer): no UX, doesn't scale

**Critical implementation notes:**
- Webhook signature verification is mandatory (verify `stripe-signature` header on every webhook call)
- Orphaned `pending_payment` bookings must be cancelled after 2 hours (background job or scheduled task)
- No Stripe Customer object in MVP (add in v2 for saved cards / subscriptions)

**Panel sign-off:** Viren (product owner) — approved for MVP scope

---

## 2026-06-04 — Center Onboarding Flow (Two-Sided Marketplace)

**Topic:** How retreat centers join the platform

**Decision:** Self-service registration + platform admin approval. Centers register as a user account (`role=center_admin`), submit an application form (name, location, description, certifications, contact), then platform admin approves or rejects in the admin queue. Approval triggers an email notification to the center, granting access to the center admin portal.

**Rationale:** Self-service lowers friction for centers wanting to list (reduces support burden). Admin approval gate ensures quality and weeds out spam / unvetted centers (trust is our positioning advantage). Email approval creates a touchpoint and can include onboarding instructions.

**Alternatives considered:**
- Instant auto-approval: speeds onboarding but sacrifices curation and brand trust
- Invitation-only: slow to scale supply side (centers can't find us)
- Hybrid (approval but with auto-approve for recognized certifications): good for v2, too complex for MVP

**Implementation:** Add `GET /list-your-retreat-center` (public landing page) and `POST /api/center-admin/apply` (application form submission). See PLAN.md Phase 7 for admin queue.

**Panel sign-off:** Viren — approved for Phase 7 build

---

## 2026-06-04 — Transactional Email Provider

**Topic:** Booking confirmations, center notifications, consultation acknowledgements

**Decision:** Resend (API-first email service). Simplest Python SDK, generous free tier (100 emails/day), excellent deliverability, integrates cleanly with FastAPI.

**Rationale:** Resend is the fastest to integrate and has zero setup friction (API key only, no SMTP server). SendGrid and Mailgun are heavier. Resend's free tier gets us through MVP launch; upgrade cost is negligible if we outgrow it.

**Emails to send:**
- Booking confirmed (Phase 5): email to seeker (confirmation number + health intake summary) + email to center (new booking + guest health data)
- Consultation submitted (Phase 4): acknowledgement email to person who submitted
- User registered (Phase 2): email verification (optional but recommended)

**Env vars:** `RESEND_API_KEY`, `EMAIL_FROM`

**Panel sign-off:** Viren — approved for Phase 5 build

---

## 2026-06-04 — Community SEO Skills Adoption

**Topic:** Use pre-built community SEO skills for Phase 3 instead of custom implementation

**Decision:** Adopt two battle-tested community SEO skills from official and community repos:
1. `seo-landing-pages` (from anthropics/skills) — SEO-optimized landing pages with JSON-LD
2. `schema-markup-generator` (from aaron-he-zhu/seo-geo-claude-skills) — JSON-LD generation for retreat listings

**Rationale:** Discovery-first platform requires strong SEO foundation. Rather than researching schema types, OG best practices, and canonical patterns from scratch, these skills provide proven implementations. Saves weeks of research, reduces errors, and ensures compliance with Google's latest schema deprecations (as of Jan 2026).

**Implementation:** Use `seo-landing-pages` skill to scaffold landing page (Phase 3, step 1). Use `schema-markup-generator` skill for retreat detail, center profile, and review pages (Phase 3, steps 3–4). Both skills available globally and in project `.claude/skills/`.

**Alternatives considered:**
- Custom implementation: more control, but higher risk of schema errors and lower SEO performance
- Third-party SEO tools (Yoast, Ahrefs): vendor lock-in, higher cost

**Panel sign-off:** Viren — approved for Phase 3 build

---

## 2026-06-04 — SEO Strategy (Discovery-First Platform)

**Topic:** Organic search positioning for retreat discovery

**Decision:** SEO is a Phase 3 requirement, not Phase 9 afterthought. All public pages (landing, search results, retreat detail, center profile) ship with:
- Meta title (< 60 chars), description (120–155 chars), OG tags (image, title, description)
- JSON-LD structured data (LodgingBusiness for centers, LodgingReservation for bookings, Review for guest reviews)
- Canonical tags on paginated search results
- `robots.txt` and `sitemap.xml` (GET `/sitemap.xml` on backend)

**Rationale:** Discovery-first positioning means organic search is not optional — it's the primary acquisition channel. People searching "panchakarma retreat Kerala" or "dosha quiz ayurveda" are the primary user segment. Meta tags and structured data are the difference between being invisible and ranking.

**Alternatives considered:**
- Defer to post-launch: costly if we're invisible at launch and lose early users to competitors
- Basic meta only (no JSON-LD): weaker click-through in search results, no rich snippets

**Quality gate:** All retreat detail pages and center profiles must have title, description, OG tags, and JSON-LD before Phase 9 deployment.

**Panel sign-off:** Viren — approved for Phase 3 build

