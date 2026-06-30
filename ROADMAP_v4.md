# _ROADMAP.md — Role Garden
**Last updated:** June 29, 2026

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
| June 29 | **Session A shipped — Stripe + Paywall.** Live Stripe account, $39/mo product, webhook configured. `/api/stripe/create-subscription` and `/api/stripe/webhook` endpoints live in proxy. `authBootGate` paywall gate live. `ob_step6` CC form functional (single-column placeholder, not yet matching two-column mock). Klaviyo `active_members` list created and wired. Verified end-to-end with real card on fresh signup — subscription created, Supabase updated immediately, Klaviyo triggered. Two bugs logged for Session B (paywall race condition on fresh signup, pre-existing analytics ReferenceError). |

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

#### Session A — Stripe + Paywall — ✅ SHIPPED June 29
- `/api/stripe/create-subscription` proxy endpoint — done, verified end-to-end
- CC form wired in `ob_step6` — done, single-column placeholder layout (two-column mock design is Session B scope)
- Webhook handler for subscription events — done (`customer.subscription.created/updated/deleted`, `invoice.payment_succeeded/failed`)
- `authBootGate` paywall check — done, fails open on Supabase error
- `active_members` Klaviyo trigger on trial activation — done, verified firing on real signup

**Post-ship fix (same day):** Webhook-only Supabase write was unreliable — RLS grants missing on `service_role`, and update matched 0 rows on first-ever subscription (`stripe_customer_id` not yet set, chicken-and-egg). Fixed by writing trial activation to Supabase immediately inside `/api/stripe/create-subscription` (by user ID, not customer ID), with the webhook write now acting as a backup/confirmation path rather than primary.

**Known gaps carried to Session B:**
- Paywall race condition on fresh signup — sometimes shows old onboarding instead of `ob_step6` on first load, resolves on hard refresh
- `ob_step6` doesn't yet match the two-column mock (`ob_step6_stripe.html`) — currently single-column
- No post-payment confirmation message — user lands silently in app after successful charge
- Stripe Link button not yet disabled on card element
- Day-7 failed payment has no automated access revocation (manual process documented in `RG_manual_tasks_guide.md`)

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

