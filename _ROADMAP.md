# Role Garden — Sprint Roadmap

> **Last updated:** Day 27-28 — Tuesday June 24, 2026 (Data infrastructure + Security + Design system + Email infrastructure)
> **Purpose:** Sprint state + session-by-session plan through V1 paid-media launch (7/6/2026).
> **Framing lock:** V1 (current build, expanding audience) + V2 (next major) only. No alpha/beta/V1.5.

---

## Sprint header

| Field | Value |
|---|---|
| **Current phase** | V1 — Phase B Experience Overhaul |
| **Day 28** | Tuesday June 24, 2026 |
| **Paid media target** | Sunday July 6, 2026 (Day 40) |
| **Pace** | ~5-7h/day solo |
| **index.html** | ~16,460 lines (post June 24 data infrastructure session) |
| **seed_jobs.js** | Day 25 A.8.5 patch (1,602 lines) |
| **proxy.js** | rg-proxy repo — Klaviyo endpoint added June 24 |
| **Pool state** | 8,848 jobs, 147 companies (post full re-seed Day 25) |

---

## Phase overview

| Phase | Days | Theme | Est. effort |
|---|---|---|---|
| **A** | 24-25 | Foundation refinement | COMPLETE |
| **B** | 26-32 | Experience overhaul | ~40-55h |
| **C** | 33-39 | Launch readiness | ~30-45h |
| **Launch** | 40 (Sun 7/6) | Paid media on | — |

---

## PHASE A — Foundation refinement
**COMPLETE — All sessions shipped Days 24-25.**
See shipped milestones audit trail below for full detail.

---

## PHASE B — Experience overhaul

**Days 26-32. Goal: V3 redesign, nightly seeds, monetization infra, data layer.**

### Data & Experimentation Infrastructure
**Status: SHIPPED Day 27-28 (June 23-24)**

**Analytics + identity (index.html +465 lines):**
- `rg_user_id` generated on first pageview via localStorage — single permanent ID
- `RG_EXPERIMENT_CONFIG` — control baseline (jd_first / 14_day_cc / $39 / assessment_first)
- `rgGetOrCreateAnonId` / `rgGetSessionId` — persistent identity
- `rgCaptureUTMs` + `deriveTrafficSource` — first-touch attribution, never overwritten
- `rgAssignCohort` — experiment variant assignment on first visit, never overwritten
- `rgSetEntryPoint` — first meaningful action wins
- `rgFirePixelEvent` — dataLayer event mapping for GTM → GA4
- `rgTrack` — unified event logger to Supabase rg_events + dataLayer
- `rgInitAnalytics` — single init call wired at boot before authBootGate
- `rgPersistCohortOnSignup` — writes cohort + attribution to users table on signup only
- `rgComputePersonaCluster` — rule-based persona bucketing
- `rgPersistResumeProfile` — writes resume-derived fields post-extraction
- `rgTriggerPresignupEmail` — calls Render proxy /api/klaviyo/presignup
- `obStep2CanProceed` / `obCaptureEmail` — email capture on resume upload step (required)

**GTM + GA4:**
- GTM container GTM-P7WVMPZT installed in index.html (head + body)
- GA4 property: Role Garden / G-Y3XBW9PR41
- 7 GTM tags published: Google Tag GA4, GA4 JD Pasted, Resume Uploaded, Opportunity Created, Lead, Signup, Trial Activated
- rg_trial_activated marked as GA4 key event ($39, once per session)

**Supabase schema (all 4 blocks verified):**
- Block A: ALTER TABLE users — 51 new columns (cohort, attribution, resume profile, funnel timestamps, trial config, legal)
- Block B: trg_set_rg_user_id trigger — auto-populates rg_user_id on INSERT, backfilled existing rows
- Block C: rg_events table + 4 indexes + RLS policies
- Block D: rg_email_captures table + indexes

**Security fix (June 24):**
- RLS enabled on all 10 public Supabase tables (was fully exposed)
- Confirmed: client uses anon key only. service_role in server-side scripts only.

**Klaviyo + pre-signup email:**
- Klaviyo account created. Audience ID: 28979d50-75e8-42ef-87ac-68e4e5dd6edb
- List: pre_signup_leads (ID: XQ9VMi)
- 5 custom properties: capture_context, homepage_variant, utm_source, traffic_source, converted
- /api/klaviyo/presignup endpoint added to proxy.js — adds contact to Klaviyo + sends Touch 1 via Resend
- Touch 1 email: "Your assessment is ready" — verified delivered ✅
- KLAVIYO_API_KEY added to Render env vars

**Open items from data infrastructure:**
1. Extend extractCareerProfile prompt — 5 missing fields (current_title, current_company, years_experience, education_level, location_raw) — IN BUILD CHAT
2. 4 funnel queries to save in Supabase SQL editor — IN BUILD CHAT
3. Fix presignup email CTA URL + remove broken unsubscribe link — IN BUILD CHAT
4. Apply brand email template to Touch 1, Confirm signup, Reset password — IN BUILD CHAT
5. Touch 2 + Touch 3 Klaviyo flows — MANUAL TASK (Marcelo)
6. Klaviyo → Meta integration — MANUAL TASK (Marcelo)
7. Klaviyo → Google Ads integration — MANUAL TASK (Marcelo)
8. GTM tags + triggers configuration (6 triggers, 7 tags) — MANUAL TASK (Marcelo)
9. rg_trial_activated as primary conversion in Meta + Google Ads — MANUAL TASK (Marcelo)

**North star metrics locked:**
- Layer 1: visitor → trial_activated (CC entered) — primary optimization target
- Layer 2: day_3_return_rate — leading indicator
- Layer 3: trial → paid — watch but don't optimize until 50+ conversions

### Design System + V3 Mockups
**Status: SHIPPED Day 28 (June 24)**

**Design system decisions locked:**
- Two layout patterns: Conversion (single column centered, no rail) vs Product (two column + right rail)
- Matches IA frozen — visual refinement only, no restructuring
- Personality: Helpful, Smart, Modern, Professional, Optimistic
- References: Airbnb, Linear, Headspace, Calm, Mercury
- Design test: "Trusted career advisor" vs "software to operate" — first = approve
- Signup framing corrected: "Start your free trial" not "see your results"
- Entry points for signup: Pursue This, Start Free Trial, Load More, Find Recommendations, Workspace CTA

**7 mockups built and approved:**
- matches_upload_v3.html — single column conversion, resume + email in one card, trust strip
- matches_assessed_v3.html — visual refinement only, IA unchanged
- matches_empty_v3.html — visual refinement only
- signup_v3.html — "Start your free trial" headline, conditional opp chip, trial callout, what happens next
- trial_v3.html — single column, feature list + timeline before CC form
- homepage_v3.html — refined, tighter spacing, harmonized tokens
- application_v3.html — not modified this phase, do not redesign

**Design brief:** NOT YET WRITTEN — next driver session item

### B.1 — V3 redesign (JD-first, two-column Matches, conversion layouts)
**Status: PENDING — design brief not yet written**
- Mockups approved, ready to brief
- Blocked on: design brief

### B.2 — Nightly seed Edge Function
**Status: PENDING**

### B.3 — Prompt security (move Haiku prompts to Render proxy)
**Status: PENDING**
- Render always-on required before B.3 ships

### B.4 — Supabase persistence (resume + career profile + job actions)
**Status: PENDING**
- P0 pre-launch. Career profile + skipped/saved/opportunity actions all localStorage-only.
- More urgent given JD-first flow — state breaks on device switch.

### B.5 — Stripe integration + trial paywall
**Status: PENDING — significant gap before paid media**
- CC/trial flow does not exist in product yet
- C.6 on roadmap but may need to move earlier given July 6 target

### B.6 — Custom system adapters: Google + Meta
**Status: PENDING**

### B.7 — OAuth: Google + Microsoft
**Status: PENDING**

### B.8 — M1A Deeper Match Analysis
**Status: PENDING**

### B.9 — M1A 5-dim → 4-dim migration
**Status: PENDING — P1 4.23**

### B.10 — Admin panel + Observability
**Status: PENDING**

---

## PHASE C — Launch readiness

**Days 33-39. Goal: monetization + legal + polish.**

### C.1 — Resume engine: structured extraction
### C.2 — Resume engine: tailoring + PDF
### C.3 — M1D Strategy Coach text-only
### C.4 — STAR library workflow revisit
### C.5 — Mobile browser experience
### C.6 — Stripe integration + pricing revisit
### C.7 — Transactional emails via Resend
**Note: Partially addressed June 24 — Confirm signup + Reset password templates redesigned. Trial-specific emails (day 7 reminder, day 13 warning, expired, payment success/failure, cancellation) remain Phase C work.**

### C.8 — Marketing site (Framer)
**Note: Homepage mockup (homepage_v3.html) built June 24 — available as design reference for Framer build.**

### C.9 — Industry expansion (12 new verticals)
**Note: P1 4.10 — INDUSTRY_BUCKET_CLUSTERS missing these verticals in frontend.**

### C.10 — ToS lawyer review
**Note: TOS updated June 24 — Section 7.2(c) added (resume data → ad audience clause). Lawyer review target July 3.**

### C.11 — Trademark filing
### C.12 — robots.txt + Cloudflare hardening
**Note: P1 4.15 — noindex removal required before July 6. robots.txt change = 2-line fix.**

### C.13 — Incident response plan

---

## Day 40 — Sunday July 6 — PAID MEDIA ON

- Meta Ads launch
- Google Ads launch
- LinkedIn Ads launch
- AdCreative.ai ($29/mo) for creative variations
- GA4 + GTM + Klaviyo fully configured
- UTM discipline enforced
- Monitor: signups, trial activation rate, day 3 return rate, trial-to-paid

---

## Open P1s (active)

| # | Issue | Status |
|---|---|---|
| 4.9 | priority_boost still in preRankJobs() — 2-line fix | Open |
| 4.10 | INDUSTRY_BUCKET_CLUSTERS missing 12 new verticals | Open — Phase C |
| 4.15 | rolegarden.com noindex — remove before paid media | Open — must fix before July 6 |
| 4.23 | M1A 5-dim vs 4-dim mismatch | Open — Phase B B.9 |
| 4.24 | Dream Companies autocomplete showing email | Open |
| 4.25 | Resume + career profile localStorage only — Supabase persistence needed | Open — Phase B B.4 |
| 4.26 | Skipped/saved/opportunity actions localStorage only | Open — Phase B B.4 |
| 4.27 | Mobile welcome overlay partial render | Open — fix in B.1 |

---

## V2 (LOCKED — post-paid-media)

| # | Item | Est. effort |
|---|---|---|
| V2.1 | Employer self-service onboarding | ~10-15h |
| V2.2 | AI agents (RG Ops Agent suite) | ~15-20h |
| V2.3 | Workday adapter | ~14-18h |
| V2.4 | Feedback tool | ~3-4h |
| V2.5 | International expansion | ~6-8h |
| V2.6 | Bundle migration (single HTML → React/Vite) | ~40-80h |

---

## Shipped milestones (audit trail)

| Day | Date | Shipped |
|---|---|---|
| 8 | May 29 | LLC filed, Google Workspace live, social pages live |
| 11 | June 2 | $39 charm pricing locked, auto-search trial+paid parity |
| 12 | June 3 | Saved searches cap 10, daily auto-search toggle, rail IA |
| 13 | June 4 | scoreJobs Haiku rewrite, 4-dim, career profile, card redesign (Day 10A). M1D V1 = text-only locked |
| 14 | June 5 | Phased launch timeline locked: 7/6 paid media |
| 17 | June 8 | Day 10A.4 polish, frameworkless prep ships |
| 18 | June 11 | Phase 1 nightly index v5.2 (337 unique rows) |
| 19 | June 12 | Browser verify Phase 1, GitHub setup |
| 20 | June 13 | Phase 2a Haiku classification at seed (v6.0, 673 inserted) |
| 22 | June 15 | Phase 2c ATS adapter framework (v7.0, Greenhouse+Lever+Ashby, 1,348 inserted) |
| 23 | June 16 | JSearch removed, mega-cluster buckets, Logo.dev, 92 ATS companies, 1,478 inserted. V1/V2 framing locked. |
| 24 | June 17 | A.1-A.5.5: workflow docs, Ashby fix, multi-role classifier, 15 title buckets, bucket-tier sort, profile extraction fix, chip hide. |
| 25 | June 18 | A.7.5-A.8: Load More fix, cache warm, Haiku title fallback, 25 buckets, remote fix. Full re-seed: 8,848 jobs, 147 companies. rolegarden.com live. |
| 26 | June 19 | A.8.6: auth timing, cascade render, Load More skeleton, onboarding loading state. Card fixes: stripHtml, batch 6→10, jobIdx, saveSkipped, skip picker removed, M1 auto-run removed, opportunity suppression. |
| 27-28 | June 23-24 | **Data infrastructure:** GTM (GTM-P7WVMPZT), GA4 (G-Y3XBW9PR41), analytics JS block (+465 lines), Supabase schema (51 columns + rg_events + rg_email_captures), RLS on all 10 tables. **Klaviyo:** pre_signup_leads list, custom properties, /api/klaviyo/presignup endpoint, Touch 1 email verified. **Design system:** layout patterns locked, personality locked, 7 V3 mockups approved. **TOS:** Section 7.2(c) added. **proxy.js:** connected to VS Code, git workflow established. |
