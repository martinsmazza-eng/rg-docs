# _ROADMAP.md — Role Garden
**Last updated:** June 27, 2026

---

## Launch Target: July 6, 2026

---

## Shipped Milestones

| Date | What shipped |
|---|---|
| May 9-21 | Core app: resume upload, job search, M1A assessment, recruiter prep, HM prep, final round prep, STAR library, Supabase integration |
| May 21 | Production domain `rolegarden.com` live. `index.html` renamed from `matchtalent_v7.html` |
| May 22 | RLS enabled on all Supabase tables. Cross-context signup bug fixed. |
| May 28 | Hardcoded credentials purged. Routing gate fixed. |
| June 4-8 | 4-dim scoring (collapsed from 5). Parallel fetch. JSearch Pro. |
| June 24 | GTM + GA4 live on app. Analytics JS block (~300 lines). 51-column Supabase schema. Pre-signup email flow (Resend + Klaviyo Touch 1A/1B). Security: RLS on all tables. |
| June 25 | JD widget removed from app. `rg_marketing_widget.html` standalone page built. `rgCheckAndRunPendingJD()` wired. Pre-signup Klaviyo flows built. |
| June 26-27 | V4 design pivot (resume-first). Full mock set (16 files) approved. 7-day member journey email spec locked. Mobile mocks built. |

---

## Current Sprint — To Launch (July 6)

### Marcelo to-do (manual, no code)
- [ ] Google Cloud Console — OAuth app → Client ID + Secret → Supabase Auth
- [ ] Microsoft Azure — App registration → Client ID + Secret → Supabase Auth
- [ ] Stripe — Create account + product + price ($39/mo) → get price ID
- [ ] Klaviyo — Build Touch 2 + 3 pre-signup flows
- [ ] Klaviyo — Create `active_members` list
- [ ] Klaviyo — Build Day 0-7 member journey (can build flow now, wire trigger after Stripe)
- [ ] Klaviyo — Connect → Meta Ads
- [ ] Klaviyo — Connect → Google Ads
- [ ] GA4 — Add second data stream for `rolegarden.com`
- [ ] GTM — Install on `rolegarden.com` (same container `GTM-P7WVMPZT`)

### Build sessions (code)

#### Session A — Stripe + Paywall (P0 — write brief first)
- `/api/stripe/create-subscription` proxy endpoint
- CC form wired in `ob_step6`
- Webhook handler for subscription events
- `authBootGate` paywall check: if no `trial_activated_at` and not `is_free_account` → redirect to Stripe
- `active_members` Klaviyo trigger on trial activation

#### Session B — V4 Onboarding + OAuth (depends on OAuth credentials + Stripe)
- 6-step standalone onboarding flow replacing modal overlay
- Google OAuth wiring (Supabase `signInWithOAuth`)
- Microsoft OAuth wiring
- Mobile reminder email (`/api/resend/reminder` proxy endpoint)
- Wire "Email me a reminder" link on ob_step2

#### Session C — V4 Design: Acquisition Funnel
- `rolegarden.com` homepage build (`homepage_v4c.html` reference)
- GTM snippet on marketing site
- Marketing site mobile responsive (768px breakpoint)
- Matches onboarding entry point wiring

#### Session D — V4 Design: Matches Page
- 3-column layout (left: Career Tracks, center: matches, right: Assess any job)
- Career Tracks move from right to left rail
- Assess any job panel: paste JD text only (no URL)
- Match card V4 redesign (full card content on all cards)

#### Session E — V4 Design: Application Workspace
- Assessment tab redesign
- Opportunity Notes in all prep views
- High risk — preserves M1 output pipeline

#### Session F — V4 Design: Prep Sessions
- Recruiter, HM, Final Round, Custom Prep
- 2-column layout, hidden left rail, opp breadcrumb
- Low-medium risk

#### Session G — V4 Design: Profile + Settings
- STAR library redesign
- Settings page redesign
- Low risk

#### Session H — Mobile App (after desktop sessions complete)
- Matches mobile: Career Tracks horizontal strip, stacked cards, Assess FAB, bottom nav
- Assessment mobile: assessment tab only, prep tabs locked "Available on desktop"
- Separate session after desktop is stable

---

## V1 Backlog (post-launch)

| ID | Item | Notes |
|---|---|---|
| V1.1 | Pending JD wire fix | `rgCheckAndRunPendingJD` — brief exists |
| V1.2 | URL fetch for ATS sources | Greenhouse, Lever, Ashby public APIs. Brief exists. |
| V1.3 | Role Garden Helper (bookmarklet) | Multi-source job clipper. Brief exists. |
| V1.4 | `extractCareerProfile` prompt extension | Return current_title, current_company, years_experience, education_level, location_raw |
| V1.5 | 4 funnel queries in Supabase | Visitor → trial conversion, day 3 return rate, feature usage, drop-off |
| V1.6 | `robots.txt` — change to `Allow: /` | Currently blocks indexing (intentional until launch) |
| V1.7 | M1A runM1 prompt update to 4-dim | Still on 5-dim |

---

## V2 Roadmap

| ID | Item |
|---|---|
| V2.1 | Behavioral email personalization (replace universal 7-day journey with behavior-triggered flows) |
| V2.2 | Auto-search / cron (daily re-run of Career Track searches, email alerts) |
| V2.3 | Path A — mobile email ingest (forward JD → get package via email) |
| V2.4 | Three-column workspace layout with right AI rail |
| V2.5 | Resume PDF rendering |
| V2.6 | JD edit on existing opportunity |
| V2.7 | Google Search Ads + sitelink extensions |
| V2.8 | SEO content strategy (`robots.txt` open, AEO-optimized FAQ pages) |
| V2.9 | Chrome extension — Role Garden Job Clipper |
| V2.10 | Multiple career tracks with independent saved searches |

---

## Build Session Reference

**References for every build chat:**
- `brand_design_system_v3.md` — tokens, components, non-negotiables
- Approved mocks — exact visual target for every screen
- `index.html` (~16,627 lines) — attach to every app build chat
- `proxy.js` (~774 lines) — attach when proxy changes needed
- `_CONSTITUTION.md` — what never to touch
- `CURRENT_STATE.md` (this file updated) — architecture reality

**Session naming convention:** `RG [Surface] V4 Build — [Date]`

