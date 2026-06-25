# Role Garden — CURRENT STATE

> **Last updated:** Day 27-28 — Tuesday June 24, 2026 (Data infrastructure + Security + Design system + Email infrastructure)
> **Purpose:** Architectural reality for build chats. Not a changelog. Not a roadmap.

---

## 1. Product framing

Role Garden is a premium AI career platform for professionals seeking their next role at exceptional companies — across industries, across functions. Single tier $39/mo with 14-day card-on-signup trial.

**Core user question:** "Is this opportunity worth pursuing?"

**JD-first acquisition funnel (locked June 24):**
Paste JD → Upload resume + email → Assessment → Matches preview → Trial gate → Signup → CC

**Two primary user modes:**
1. **Matches** — Assess opportunities, see recommendations (JD-first entry)
2. **Applications** — Full workspace per opportunity (resume, cover letter, outreach, interview prep)

---

## 2. Tech stack

| Component | Detail |
|---|---|
| **Frontend** | Single-file vanilla JS HTML SPA at `app.rolegarden.com` (~16,460 lines) |
| **Hosting** | Netlify — auto-deploys from GitHub `martinsmazza-eng/deploy_rg` |
| **Proxy** | Render (always-on $7/mo) at `rg-proxy-flpd.onrender.com` — Claude API, Resend, Klaviyo |
| **DNS** | Cloudflare (apex + send.rolegarden.com) |
| **Database** | Supabase Pro — `https://xveicdwpxwbezwncbxbp.supabase.co` |
| **Email (transactional)** | Resend via `send.rolegarden.com` |
| **Email (marketing/nurture)** | Klaviyo — pre_signup_leads list (ID: XQ9VMi), Audience ID: 28979d50-75e8-42ef-87ac-68e4e5dd6edb |
| **Tag management** | Google Tag Manager — GTM-P7WVMPZT |
| **Analytics** | GA4 — G-Y3XBW9PR41 |
| **LLM** | Anthropic API Tier 2. Haiku ~90%. Sonnet for M1D + Frameworkless Prep only |
| **Marketing site** | `rolegarden.com` — Netlify, noindex holding page. Framer build Phase C. |
| **Auth** | Email/password via Supabase. OAuth (Google + Microsoft) = Phase B work |

**Local working directories:**
- `~/Documents/deploy_rg/` — frontend repo
- `~/rolegarden_seed/` — seed repo
- `~/Documents/rg-proxy/` — proxy repo (git connected to GitHub June 24)

**GitHub repos:**
- `martinsmazza-eng/deploy_rg` — frontend
- `martinsmazza-eng/rolegarden_seed` — seed
- `martinsmazza-eng/rg-proxy` — Render proxy (proxy.js — Klaviyo endpoint added June 24)
- `martinsmazza-eng/rolegarden-marketing` — marketing homepage

**Domains:**
- `rolegarden.com` — marketing (Netlify, noindex — remove before July 6)
- `app.rolegarden.com` — app (Netlify)
- `send.rolegarden.com` — Resend transactional
- `rg-proxy-flpd.onrender.com` — Render proxy

---

## 3. Data & experimentation infrastructure (SHIPPED June 24)

### 3.1 Identity model
- `rg_user_id` = `rg_usr_<uuid>` generated on first pageview, stored in localStorage
- Same ID becomes permanent Supabase `users.id` mirror at signup
- Pre-signup events tracked via `anon_id` (same value as `rg_user_id` in localStorage)

### 3.2 Experiment config (control baseline — never hardcode variants)
```javascript
RG_EXPERIMENT_CONFIG = {
  homepage_variant: ['jd_first'],
  trial_variant:    ['14_day_cc'],
  pricing_variant:  ['39'],
  product_variant:  ['assessment_first']
}
```
Assignment is random on first visit, stored in localStorage, never overwritten.

### 3.3 GTM dataLayer events
| Event | Purpose | Ad platform |
|---|---|---|
| rg_jd_pasted | Intent signal | Audience building |
| rg_resume_uploaded | Commitment signal | Audience building |
| rg_opportunity_created | Pursue This clicked | Secondary signal |
| rg_lead | Email captured | Secondary signal |
| rg_signup | Account created | Funnel data only |
| rg_trial_activated | CC entered | **Primary conversion** |

### 3.4 North star metrics
- Layer 1: `visitor → trial_activated` — optimize against this first
- Layer 2: `day_3_return_rate` — leading indicator for conversion
- Layer 3: `trial → paid` — watch, don't optimize until 50+ conversions

### 3.5 Supabase schema additions (June 24)
Users table: 51 new columns — rg_user_id, cohort (homepage_variant, trial_variant, pricing_variant, product_variant, entry_point), attribution (utm_source/medium/campaign/content/term, landing_page, referrer_url, traffic_source), resume-derived profile (current_title, current_company, years_experience, seniority_level, functions, skills, education_level, location_raw, persona_cluster, resume_uploaded_at), funnel timestamps (first_seen_at, trial_activated_at, converted_at, cancelled_at, churned_at), trial config (trial_days, trial_cc_required, trial_variant_price), legal (tos_accepted_at, tos_version).

New tables: `rg_events` (event log, all pre/post signup), `rg_email_captures` (pre-signup lead capture with attribution).

Trigger: `trg_set_rg_user_id` — auto-populates `rg_user_id = 'rg_usr_' || id` on INSERT.

RLS: All 10 public tables have RLS enabled (fixed June 24 — was fully exposed).

---

## 4. Search engine architecture

### 4.1. Pool sourcing — 100% ATS-direct

| Provider | Companies | Notes |
|---|---|---|
| **Greenhouse** | 63 | Stripe, Datadog, MongoDB, Anthropic, HubSpot, etc. |
| **Lever** | 12 | Klaviyo, Outreach, Aircall, Tinuiti, ContentSquare |
| **Ashby** | 19 | OpenAI, Notion, Plaid, Ramp, Vanta, Cohere, Sentry, etc. |

**Pool state:** 8,848 jobs, 147 companies (post full re-seed Day 25).

### 4.2. Seed pipeline (seed_jobs.js — 1,602 lines, Day 25 A.8.5)
1. Load curated companies from `rg_curated_companies` where `ats_provider IS NOT NULL`
2. JS slug dedup — prevents multi-bucket companies from being processed twice
3. For each company → adapter fetches jobs
4. US-only location filter + non-US remote exclusion at adapter level
5. Title pre-filter — 28 universal exclusions
6. Each job → Haiku classification → 25-bucket industry assignment + role_category title fallback
7. Drop if: `reject_other` OR `role_match_no` OR `confidence < 0.6` OR `classification_failed`
8. `assignTitleBucket(title, company)` → 15-bucket regex; null falls back to Haiku `role_category`
9. Insert with `priority_boost = true` if classified bucket matches curated bucket
10. `expires_at = now() + 60 days` on every insert

### 4.3. Frontend search flow
1. `runM0Search` fires on load or chip change
2. `fetchFromIndex` queries `rg_jobs_index` (Supabase)
3. `preRankJobs` — client-side sort by bucket match tier (Tier 2 = exact, Tier 1 = adjacent, Tier 0 = other)
4. `scoreJobs` — Haiku 4-dim scoring (Industry 30%, Role 30%, Seniority 20%, Skills 20%)
5. Cards render via cascade (batch 10, pre-score next batch in background)

### 4.4. Scoring dimensions (4-dim, locked Day 13 / Day 10A)
- **Industry fit** (0-30 pts, gate): match=50 base, adjacent=45, mismatch=hard cap 39
- **Role alignment** (0-30 pts)
- **Seniority match** (0-20 pts)
- **Skills fit** (0-20 pts, only if industry passed)

---

## 5. Email infrastructure (SHIPPED June 24)

### 5.1 Transactional (Resend + Supabase Auth)
| Email | Where | Status |
|---|---|---|
| Confirm sign up | Supabase Auth template | Brand template — IN BUILD CHAT |
| Reset password | Supabase Auth template | Brand template — IN BUILD CHAT |
| Touch 1 presignup | proxy.js /api/klaviyo/presignup | Shipped — brand template IN BUILD CHAT |

### 5.2 Marketing/nurture (Klaviyo)
| Email | Status |
|---|---|
| Touch 1 — "Your assessment is ready" | Shipped via Resend from Klaviyo endpoint |
| Touch 2 — Day 2 — "One opportunity worth a look" | Manual task — build in Klaviyo flow builder |
| Touch 3 — Day 5 — "Still thinking it over?" | Manual task — build in Klaviyo flow builder |

### 5.3 Klaviyo configuration
- Audience ID: 28979d50-75e8-42ef-87ac-68e4e5dd6edb
- List: pre_signup_leads (XQ9VMi)
- Properties: capture_context, homepage_variant, utm_source, traffic_source, converted
- Exit condition for flows: converted = true → exit immediately
- Meta + Google integrations: PENDING (manual task)

---

## 6. Design system (locked June 24)

### 6.1 Layout patterns
**Conversion flows** (resume upload, signup, CC/trial):
- Single column, centered, max-width 480-560px
- Minimal nav: logo + step pips + contextual link
- No right rail, no secondary nav

**Product experience** (Matches, Applications):
- Two column: main + right rail
- IA frozen — no restructuring

### 6.2 Brand tokens
```
--green:      #1B6B3A
--green-dark: #0E3D2C
--cream:      #F5F1EA
--border:     #E8E4D8
--ink:        #1A2E22
--ink-mid:    #5C6B62
--ink-muted:  #7A887E
--amber:      #D8923A
Font: Inter
```

### 6.3 V3 mockups (approved June 24)
All in outputs folder:
- `matches_upload_v3.html` — conversion layout, resume + email, trust strip
- `matches_assessed_v3.html` — product layout, visual refinement
- `matches_empty_v3.html` — product layout, visual refinement
- `signup_v3.html` — "Start your free trial", conditional opp chip
- `trial_v3.html` — feature list + timeline before CC form
- `homepage_v3.html` — refined visual language
- `application_v3.html` — not modified this phase

### 6.4 Design brief
NOT YET WRITTEN — next driver session item.

---

## 7. What's broken (active P0/P1 list)

| # | Issue | Severity | Status |
|---|---|---|---|
| 4.9 | priority_boost still in preRankJobs() — 2-line fix | P1 | Open |
| 4.10 | INDUSTRY_BUCKET_CLUSTERS missing 12 new verticals | P1 | Open — Phase C |
| 4.15 | rolegarden.com noindex — remove before paid media | P1 | Open — must fix before July 6 |
| 4.23 | M1A 5-dim vs 4-dim mismatch | P1 | Open — Phase B B.9 |
| 4.24 | Dream Companies autocomplete showing email | P1 | Open |
| 4.25 | Resume + career profile localStorage only | P0 pre-launch | Open — Phase B B.4 |
| 4.26 | Skipped/saved/opportunity actions localStorage only | P1 pre-launch | Open — Phase B B.4 |
| 4.27 | Mobile welcome overlay partial render | P1 | Open — fix in B.1 |
| 4.28 | Supabase funnel queries not yet saved | P1 | IN BUILD CHAT |
| 4.29 | extractCareerProfile missing 5 fields | P1 | IN BUILD CHAT |
| 4.30 | Presignup email CTA URL has no context parameters | P1 | IN BUILD CHAT |
| 4.31 | Supabase Auth email templates use default Supabase design | P1 | IN BUILD CHAT |
| 4.32 | Stripe integration does not exist — CC/trial flow not built | P0 pre-launch | Open — Phase B B.5 (may need to move earlier) |
| 4.33 | Klaviyo Touch 2 + Touch 3 flows not built | P1 | Open — manual task |
| 4.34 | GTM tags + triggers not configured | P1 | Open — manual task |
| 4.35 | Klaviyo → Meta + Google integrations not connected | P1 | Open — manual task |

---

## 8. Locked decisions (do not relitigate)

- **$39/mo single tier**, 14-day card-on-signup trial, full feature parity
- **Auto-search enabled** in trial + paid
- **Haiku ~90%** — Sonnet only for M1D + Frameworkless Prep
- **Role Garden Lite $9/mo** — cancellation-save offer only, not publicly marketed
- **JSearch removed permanently** — ATS-direct only
- **V1/V2 framing only** — no alpha/beta/V1.5
- **M1D V1 scope** — text-only Strategy Coach, no deck export
- **JD-first acquisition funnel** — paste JD → resume upload → assessment → matches → trial gate → signup → CC
- **Signup trigger** — assessment result visible before signup. User signs up to ACT, not to SEE results.
- **Single rg_user_id** — generated first pageview, permanent, no separate visitor table
- **GTM for all tags** — no hardcoded pixel snippets. One GTM container, all tags via GTM interface.
- **Klaviyo for pre-signup nurture** — Resend for transactional only
- **Matches IA frozen** — two column, right rail = Career Tracks. No restructuring in V3.
- **Application Workspace** — not redesigned in V3 phase. Inherits Matches visual language later.
- **Bundle migration** (single HTML → React/Vite) — V2 work

---

## 9. Source-of-truth document priority

1. **`CURRENT_STATE.md`** (this doc) — architectural reality
2. **`_ROADMAP.md`** — sprint state, session history
3. **`driver_chat_operating_manual.md`** — process discipline
4. **`_CONSTITUTION.md`** — non-negotiable rules
5. **`BRIEF_TEMPLATE.md`** — build chat brief structure
6. **`pricing_model_locked.md`** — pricing decisions
