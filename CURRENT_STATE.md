# CURRENT_STATE.md — Role Garden
**Last updated:** July 1, 2026

---

## The Product

Role Garden is an AI career platform that matches professionals to curated opportunities, evaluates fit, and prepares them for every stage of the application and interview process.

**Positioning:** Career decision tool, not a job board.
**Tagline:** "Cultivate your career."
**Primary CTA:** "Unlock My Career Workspace"

---

## The Stack

### Repos
| Repo | Branch | Purpose |
|---|---|---|
| `martinsmazza-eng/deploy_rg` | main | App — `app.rolegarden.com` |
| `martinsmazza-eng/rg-proxy` | main | Render proxy — `rg-proxy-flpd.onrender.com` |
| `martinsmazza-eng/rolegarden-marketing` | main | Marketing site — `rolegarden.com` |

### File state
| File | Location | Lines | Notes |
|---|---|---|---|
| `index.html` | `deploy_rg` | ~17,799 | Single-file vanilla JS SPA |
| `proxy.js` | `rg-proxy` | ~774 | Node/HTTP (no Express) |
| `rg_marketing_widget.html` | `rolegarden-marketing` | ~307 | Standalone JD-first landing page (noindex) |

### Infrastructure
| Service | Config | Status |
|---|---|---|
| Netlify | `app.rolegarden.com` | ✅ Live |
| Render | `rg-proxy-flpd.onrender.com` — $7/mo Starter, always-on | ✅ Live |
| Supabase | Pro tier — `xveicdwpxwbezwncbxbp.supabase.co` | ✅ Live |
| Cloudflare | DNS for rolegarden.com | ✅ Live |
| Resend | `hello@rolegarden.com` — transactional only | ✅ Live |
| Klaviyo | Nurture sequences + ad sync | ✅ Connected, flows in progress |
| Anthropic | Claude Sonnet 4.5 (prep) + Haiku 4.5 (scoring) via proxy | ✅ Live |
| GTM | `GTM-P7WVMPZT` — on app.rolegarden.com only | ✅ Live |
| GA4 | `G-Y3XBW9PR41` — app.rolegarden.com only | ✅ Live |
| Google OAuth | Wired June 29 — Client ID/Secret in Supabase | ✅ Live |
| LinkedIn OAuth (OIDC) | Wired June 30 — Client ID/Secret in Supabase, redirect URI set | ✅ Live |
| Microsoft OAuth | Blocked on Azure tenant issue, deprioritized for LinkedIn | ❌ Pending — revisit this week, not launch-blocking |
| Stripe | Live, $39/mo product, 7-day trial wired | ✅ Live |

---

## Acquisition Strategy — V4 (LOCKED June 2026)

**Primary path:** Resume-first
```
rolegarden.com (brand page)
    ↓ "Find My Opportunities" CTA
ob_step1: Email capture (pre-auth, fires Klaviyo Touch 1A if no resume)
    ↓
ob_step2: Resume upload (no skip)
    ↓
ob_step3: Loading / personalization (premium animated screen, 15-20s)
    ↓
ob_step4: Results preview (2 cards, "See why this fits you" text link, single CTA)
    ↓ "Unlock My Career Workspace"
ob_step5: Signup — identity only (Google + Microsoft + Email, no pricing)
    ↓
ob_step6: Stripe — billing only (7-day trial, CC required, $39/month after)
    ↓
Matches page (full access)
```

**Secondary path (landing page only):** JD-first
- `rolegarden.com/rg_marketing_widget.html` — standalone noindex page
- Used for paid media campaigns targeting users with a specific opportunity in mind
- Same funnel entry — stores `rg_pending_jd` in localStorage, redirects to signup

**No access without:** resume upload + payment (exception: `is_free_account = true` in Supabase, manual toggle)

---

## Pricing Model

| Variant | Config | Status |
|---|---|---|
| 7-day trial + CC | User gets 7 days free, charged $39 on day 8 | **Default (locked)** |
| Charge immediately | User charged $39 at signup | A/B test variant |

Controlled via `trial_variant` in `RG_EXPERIMENT_CONFIG`.

Free access exception: `is_free_account` boolean in `users` table. Manual Supabase toggle. Still requires resume upload.

---

## App Architecture

### Authentication
- Supabase Auth (email + password)
- Google OAuth: ✅ wired June 29. Cloud project `role-garden` created under `rolegarden.com` org. OAuth consent screen configured (External, support email `contact@rolegarden.com` — new Restricted Google Group). Client ID + Secret created, wired into Supabase Auth providers. Verified enabled.
- LinkedIn OAuth (OIDC): ✅ wired June 30. App created under existing Role Garden LinkedIn Company Page, company verification completed. "Sign In with LinkedIn using OpenID Connect" product added (openid, profile, email scopes). Client ID + Secret wired into Supabase Auth providers, redirect URI configured on both sides. Verified enabled. Note: LinkedIn OAuth provides authentication + basic profile only — does NOT provide resume-equivalent work history/skills data (LinkedIn restricts that to Partner Program access). Does not replace resume upload flow.
- Microsoft OAuth: ❌ blocked — Azure tenant mismatch error (`AADSTS50020`) when signing into Azure portal with personal Microsoft account. Needs resolution before Session B can wire it. Not deferred — revisit this week. **Decision June 30: deprioritized in favor of LinkedIn as third sign-in option — better audience fit for job seekers. Microsoft still worth fixing eventually but no longer blocking Session B.**
- Password reset: ✅ working
- Email confirmation: ❌ disabled June 30 — was blocking session creation on signup (see Known Bugs / Decisions below). Sessions now issue immediately on `signUp()`. CC requirement at Step 6 is the real anti-fraud gate; email confirmation was not adding meaningful protection at this stage.

### Onboarding flow — LEGACY (modal overlay, still in code, not the active path)
- `showOnboardingPage()` → modal overlay — superseded by V4 standalone flow below, but NOT deleted (flagged for future cleanup)
- Mapped in full June 30 (Session B.1 summary): `obCaptureEmail()`, `obStep2CanProceed()`, `obShowStep(n)`, `obClearFile()`, `obFinish()`, `onAssessmentComplete()` — all still functional in isolation, independently read/write `rg_captured_email` and `rg_email_captures`, fire their own Klaviyo Touch 1B. Not routed to from anywhere active (`authBootGate`'s no-session branch defaults to the V4 flow). Safe to delete only as its own dedicated cleanup session — has real dependencies that need to be traced out first.

### Onboarding flow — V4 ACTIVE (shipped, in progress)
**5 steps, not 6** — standalone email-capture step eliminated June 30 (Session B.1). Order: **Signup → Resume Upload → Loading/Matching → Results → CC (Stripe)**. Account creation moved to Step 1 specifically to fix a real bug — Steps 3/4 need an authenticated Supabase session to call `/api/claude` and query `rg_jobs_index` (both RLS-protected), and originally ran pre-auth, causing 401s. Confirmed via browser testing, not just code review.

| Step | Function | Status |
|---|---|---|
| 1 | `showObStep1()` (was Step 5's signup UI, renamed) | ✅ Working — Google/LinkedIn/Email signup |
| 2 | `showObStep2()` | ✅ Working — resume upload, no skip |
| 3 | `showObStep3()` / `_obStep3Run()` | ⚠️ Functionally works (real auth, real data), but visual/content far short of `ob_step3_loading.html` mock (missing milestone richness, rotating insights, observation banner) — fix in progress |
| 4 | `showObStep4()` | ⚠️ Functionally works (real match cards with live scoring data), but missing most of `ob_step4_results.html`'s content (hero, two-column reasoning, fit summary, journey illustration, checklist, trust strip) — fix in progress |
| 5/6 | `showObStep6()` | ⚠️ Two-column layout matches mock, but payment form is missing Name on Card, Billing ZIP, and CVC is not visibly rendering — root cause not yet diagnosed, fix in progress |

Old `showObStep1` (email-only capture) renamed to `_DEPRECATED_obStep1EmailCapture`, kept in file, zero active callers.

### Matching engine
- Job index: `rg_jobs_index` table in Supabase (~8,848 jobs, ATS-direct)
- Scoring: 4-dim Haiku (Industry 30%, Role 30%, Seniority 20%, Skills 20%)
- `runM0Search()` → `scoreJobs()` → card render
- Labels: Strong Match, Very Good Match, Good Match, Fair Match (amber, min 50% bar)
- No numeric scores, no red states

### Matches page (current in code)
- 2-column: main (assess panel + results) + right rail (Career Tracks)
- Assess panel: paste JD text + URL detection
- Career Tracks: right rail

### Matches page (V4 target)
- 3-column: left (Career Tracks) + center (matches) + right (Assess any job — text only, no URL)
- Career Tracks move from right to left rail

### Application workspace
- Assessment tab: M1A (Haiku scoring + why/challenge/fit/recommendation/salary)
- Resume, Cover Letter, Outreach tabs: ✅ working
- Opportunity Notes: ✅ persistent, auto-saved, shared across prep tabs

### Prep sessions
- Recruiter Prep: ✅ 6 sections, accordion
- HM Prep: ✅ Phase 1 (STAR selection) + Phase 2 (generation)
- Final Round Prep: ✅ (sales deck / discovery / outreach)
- Custom Prep: ✅ frameworkless
- All prep: 2-column layout (center + copilot), left pipeline rail hidden, opp breadcrumb at top

### Profile + Settings
- STAR Stories library: ✅
- Account, Billing, Notifications settings: ✅

---

## Mobile (V4 scope — locked)

### Marketing site (`rolegarden.com`)
- Fully responsive — same content as desktop
- Built in same session as desktop homepage build
- Breakpoint: 768px

### App (`app.rolegarden.com`)
- **Matches:** Career Tracks as horizontal scrollable strip, stacked match cards, Assess a Job FAB button, bottom nav (Matches + Applications)
- **Assessment:** Back breadcrumb in nav, subnav tabs (Assessment active, Recruiter/HM/Final greyed "Available on desktop"), Assessment content, Pursue CTA fixed at bottom
- **Excluded from mobile:** Recruiter Prep, HM Prep, Final Round Prep, Custom Prep, Profile/STAR library
- **Settings:** simplified or excluded
- Built as separate session after desktop V4 ships

---

## Email Infrastructure

| Email | Trigger | Tool | Status |
|---|---|---|---|
| Signup confirmation | Supabase Auth | Supabase | ✅ |
| Password reset | Supabase Auth | Supabase | ✅ |
| Touch 1A — "Your assessment is waiting" | Email captured, no resume | Resend via proxy | ✅ Built |
| Touch 1B — "Your matches are ready" | Resume uploaded, no signup | Resend via proxy | ✅ Built |
| Mobile reminder — "Finish on desktop" | "Email me a reminder" clicked | Resend via proxy | ❌ Not built |
| Touch 2 — Day 2 nurture | 2 days after email capture, no signup | Klaviyo | ❌ Not built |
| Touch 3 — Day 3 nurture | 3 days after email capture, no signup | Klaviyo | ❌ Not built |
| Day 0 Welcome | Trial activated | Klaviyo `active_members` | ❌ Not built |
| Day 1-7 Member Journey | Daily after Day 0 | Klaviyo `active_members` | ❌ Not built |

### Klaviyo lists
- `pre_signup_leads` (ID: XQ9VMi) — ✅ exists. Touch 1A/1B sending. Touch 2/3 pending.
- `active_members` — ❌ not created. Triggered by `trial_activated_at` (depends on Stripe).

---

## Analytics & Tracking

### GTM / GA4
- GTM `GTM-P7WVMPZT`: ✅ on `app.rolegarden.com`
- GA4 `G-Y3XBW9PR41`: ✅ on `app.rolegarden.com`
- GTM on `rolegarden.com`: ❌ not installed
- GA4 second data stream for `rolegarden.com`: ❌ not set up

### Tracked events (app)
| Event | Purpose |
|---|---|
| `rg_jd_pasted` | Intent signal |
| `rg_resume_uploaded` | Commitment signal |
| `rg_opportunity_created` | Pursue This clicked |
| `rg_lead` | Email captured |
| `rg_signup` | Account created |
| `rg_trial_activated` | CC entered — **primary conversion** |

### Experiment config (`RG_EXPERIMENT_CONFIG`)
```javascript
homepage_variant: ['resume_first']  // was jd_first
trial_variant:    ['7_day_cc']      // was 14_day_cc
pricing_variant:  ['39']
product_variant:  ['assessment_first']
```

---

## Supabase Schema

### Key columns added June 24 (51 new columns)
- Cohort + attribution (UTM, traffic source, entry point)
- Resume profile (current_title, current_company, years_experience, seniority_level, functions, skills, education_level, location_raw, persona_cluster)
- Funnel timestamps (email_captured_at, resume_uploaded_at, trial_activated_at, converted_at)
- Trial config (trial_variant, trial_ends_at, is_free_account)
- Legal (tos_accepted_at, tos_version)

### Key tables
| Table | RLS | Notes |
|---|---|---|
| `users` | ✅ owner-only | Primary user record |
| `resumes` | ✅ owner-only | Resume files + extracted text |
| `opportunities` | ✅ owner-only | Pursued opportunities |
| `star_stories` | ✅ owner-only | STAR library |
| `rg_jobs_index` | ✅ authenticated read | 8,848 ATS-direct jobs |
| `rg_events` | ✅ owner-write | Analytics events |
| `rg_email_captures` | ✅ owner-write | Pre-signup email capture |

---

## Open Infrastructure Gaps (P0/P1)

### P0 — blocks launch
| Item | Description |
|---|---|
| V4 onboarding flow in code | 6-step flow exists as mocks only — ob_step6 (CC form) is live but not yet wired into the ob_step1-5 sequence |
| Matches 3-column layout | Career Tracks still on right rail |

**Stripe integration — SHIPPED June 29 (Session A).** CC form (ob_step6, single-column placeholder — full two-column mock design pending Session B), `/api/stripe/create-subscription` and `/api/stripe/webhook` proxy endpoints, paywall gate in `authBootGate`. Verified end-to-end with real card on fresh signup: subscription created, Supabase `users` row updated immediately (not dependent on webhook), Klaviyo `active_members` triggered. Test subscriptions cancelled post-verification.

### P1 — before July 6
| ID | Item |
|---|---|
| 4.36 | ~~Google OAuth — not wired~~ ✅ DONE June 29 |
| 4.37 | Microsoft OAuth — blocked on Azure tenant mismatch (`AADSTS50020`), personal MS account can't access Entra ID. Deprioritized June 30 in favor of LinkedIn. Revisit this week if time allows — not launch-blocking. |
| 4.47 | ~~LinkedIn OAuth — not wired~~ ✅ DONE June 30 |
| 4.38 | Mobile reminder email (`/api/resend/reminder`) |
| 4.39 | Klaviyo `active_members` list + Day 0-7 journey |
| 4.40 | Klaviyo Touch 2 + 3 pre-signup flows |
| 4.41 | GA4 second data stream for rolegarden.com |
| 4.42 | GTM on rolegarden.com |
| 4.43 | V4 visual redesign not built in code |
| 4.44 | Paywall gate in authBootGate |
| 4.45 | Mobile responsive — marketing site |
| 4.46 | Mobile app experience build (separate session after desktop) |

---

## Open Items Surfaced June 30

- **Privacy Policy + ToS not live.** Only a placeholder draft clause exists (`tos_resume_consent_clause_draft.md`), not a finalized lawyer-reviewed policy. Footer links on `homepage_v4c.html` point to `#`. Per `RG_manual_tasks_guide.md`, lawyer engagement targeted Days 22-26, finalized pages by July 3 — still on original schedule, just not done. Surfaced while creating LinkedIn OAuth app (Privacy Policy URL field, ended up not required).
- **LinkedIn Company Page copy updated June 30** — replaced "AI job-search agent" framing (violated brand manifesto's "AI stays in the background" voice rule) with manifesto-aligned About text centered on "career execution platform" and Discover/Decide/Prepare/Win messaging hierarchy. Same update should be applied to Facebook Page (same outdated copy from June 29 setup) and any other surface still using old positioning.

## Known Bugs

| Bug | Severity | Status |
|---|---|---|
| Paywall race condition on fresh signup | Resolved | Root cause confirmed June 30 (Session B.1): old `showOnboardingPage()` modal was firing unconditionally via `checkOnboarding()` even after the V4 flow's own routing took over. Fixed with an `rg_v4_onboarding_active` localStorage guard. |
| `rgInitAnalytics` ReferenceError (broad) | Resolved | Session B added a `typeof supabase === 'undefined'` guard. **Narrower version still occurs** — `rgTrack` calling `supabase.auth.getUser()` can still throw when `supabase` is technically defined but `.auth` isn't ready (different failure mode the original guard didn't cover). Non-fatal — does not block downstream calls. Not yet fixed, low priority. |
| Stripe Link button on ob_step6 | Resolved | Disabled via Stripe Dashboard → Settings → Payment Methods → Link (toggle off), June 30. More durable than the CSS fallback Session B's build chat had used as a stopgap. |
| `appShell` DOM id mismatch (3 call sites) | Resolved | `showObStep6()` and shared step helpers referenced nonexistent `appShell` id — should have been `mainArea`. Null-guarded so it never threw, but was a real latent bug. Fixed June 30 (pure rename, 3 sites). |
| `showObStep5` stale reference | Resolved | Session B.1's Item 1 rename (`showObStep5`→`showObStep1`) missed one call site — Step 4's "Unlock My Career Workspace" button still called the old name, blocking 100% of users from reaching payment. Caught in browser testing immediately post-handoff, hotfixed same day. Root cause: rename-in-place edits need a full-file grep for the old name afterward, not just a definitions check — now a standing process note for future sessions. |
| Email confirmation blocking session creation | Resolved | Supabase's "Confirm email" setting (Authentication → Providers → Email) was ON, meaning `signUp()` returned no usable session until the user clicked a confirmation link — this was the actual root cause of the Step 3/4 401 errors, not just step-ordering. Disabled June 30. Tradeoff accepted: no automatic email-ownership verification at signup, but the Stripe CC requirement at Step 6 is a stronger fraud barrier regardless. Standard SaaS practice (per industry data) is to delay or skip pre-activation email verification — this aligns with that. |
| Step 3 (loading) content gap vs mock | Open | Functionally correct (real auth, real data) but visually far short of `ob_step3_loading.html` — missing most of the milestone/animation richness. In progress, Session B.3. |
| Step 4 (results) content gap vs mock | Open | Real match cards render with live scoring data, but missing most of `ob_step4_results.html`'s content — hero section, two-column reasoning, fit summary, journey illustration, benefits checklist, trust strip. Session B's original claim that this was "confirmed feasible and built" was accurate about data flow but materially overstated relative to mock content depth. In progress, Session B.3. |
| Step 6 (CC form) missing fields | Open | Two-column layout matches mock, but the payment form is missing "Name on card" and "Billing ZIP" fields entirely, and CVC is not visibly rendering in the Stripe Card Element (root cause not yet diagnosed — could be CSS sizing or an Elements option issue). In progress, Session B.3. |

| Bare `supabase` global reference — 6 call sites | Resolved July 1 | `supabase` was never a declared variable anywhere in the file — only `window.supabase` (raw SDK) and `_supabaseClient`/`getSupabase()` (actual client) exist. Six functions used the bare name: `extractCareerProfile`, `rgTrack`, `rgPersistResumeProfile`, cohort-persistence (~L2379), `obCaptureEmail` (legacy flow), `onAssessmentComplete` (legacy flow). Session B's "fix" to `rgTrack` checked `typeof supabase === 'undefined'` — wrong pattern, since `supabase` as a bare name can resolve as `undefined` or throw a ReferenceError depending on browser scoping. All six now use `getSupabase()`. |
| `rg_events` 403 on insert | Open | RLS policy missing for `authenticated` role on `rg_events` table. Analytics events fail silently. Non-fatal — does not block any user-facing flow. Fix: add `GRANT INSERT ON public.rg_events TO authenticated` in Supabase SQL Editor, plus verify RLS policy allows authenticated user INSERT. |
| GA4 event tracking gap in new onboarding flow | Open, not yet investigated | `rg_lead` fires from the OLD Step 1 (`obStep1Continue`, deprecated by Session B.1's reorder) — likely no longer fires at all in the active flow. `rg_signup` not found firing anywhere in a grep of the pre-B.1 file. `rg_resume_uploaded` appears to still fire correctly (Step 2 stayed in the active chain). Not investigated against the actual deployed B.1 file yet — flagged June 30, to be checked after Session B.3 ships. Conversion funnel analytics may currently be incomplete/broken for the new 5-step flow. |

- **Resume text has no Supabase persistence anywhere in the codebase** (old modal flow or new V4 flow) — lives in `rg_resume_primary` localStorage only. This is why a user signing in on a different device sees "no resume uploaded." Derived profile fields (skills, seniority, persona_cluster, etc.) DO persist correctly to the `users` table via `rgPersistResumeProfile()`. **Decision: build a real `resumes` table (RLS-protected, owner-only, same trust pattern as other tables) in Session B.2.** Not deferred indefinitely, not parked as V1.5 — scoped as its own near-term session.
- **Onboarding order changed: Signup now precedes Resume Upload** (was the reverse in the original V4 spec). Two reasons, both real: (1) architectural — Steps 3/4 need an authenticated session for RLS-protected queries, and (2) product/trust — no mainstream platform (LinkedIn, Indeed, ZipRecruiter, Otta, Teal, Huntr) asks for resume upload before account creation; it's sensitive personal data and users expect an authenticated container before handing it over. Standalone email-capture step (old Step 1) eliminated — folded into the new Step 1 signup screen.

## Mock Inventory (approved, June 2026)

### Marketing site
- `homepage_v4c.html` — brand page, 8 sections, FAQ accordion, footer

### Onboarding (6 steps — conversion layout, cream bg, no app chrome)
- `ob_step1_email.html` — email capture
- `ob_step2_resume.html` — resume upload, mobile reminder link
- `ob_step3_loading.html` — animated personalization screen
- `ob_step4_results.html` — 2 match cards + product reveal + dual CTA
- `ob_step5_signup.html` — identity only (Google + Microsoft + Email)
- `ob_step6_stripe.html` — 7-day trial billing

### App — Matches
- `matches_v4b.html` — 3-column desktop

### App — Mobile
- `matches_mobile_v4.html` — Career Tracks strip, stacked cards, Assess FAB
- `assessment_mobile_v4.html` — assessment tab, locked prep tabs

### App — Application Workspace
- `application_v3.html` — assessment tab
- `recruiter_prep_v3.html` — 2-column, opp breadcrumb, Opportunity Notes
- `hm_prep_v3.html` — Phase 1 STAR selection
- `final_round_v3.html` — 2-column
- `custom_prep_v3.html` — frameworkless prep
- `profile_v3.html` — STAR library
- `settings_v3.html` — account/billing/notifications

### Marketing widget (standalone)
- `rg_marketing_widget_v3.html` — V3 styled, JD-first landing page

---

## Design System — Locked (brand_design_system_v3.md)

| Token | Value |
|---|---|
| Brand green | `#1B6B3A` |
| Green dark (header) | `#0E3D2C` |
| Cream (background) | `#F5F1EA` |
| Border | `#E8E4D8` |
| Ink | `#1A2E22` |
| Amber | `#D8923A` |
| Font | Inter only |
| Logo (light bg) | `rolegarden_logo_square_512.png` |
| Logo (dark bg) | `icon-white.png` |

### Layout patterns
- **Conversion:** single column, max-width 480-560px, cream bg, logo top left (not linked), progress pips top right, no navigation
- **Product desktop:** 3-column (left rail + center + right rail)
- **Product mobile:** single column, horizontal strips, bottom nav

