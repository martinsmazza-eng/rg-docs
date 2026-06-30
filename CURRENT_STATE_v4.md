# CURRENT_STATE.md — Role Garden
**Last updated:** June 30, 2026

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
| `index.html` | `deploy_rg` | ~16,627 | Single-file vanilla JS SPA |
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
- Email confirmation: ✅ working

### Onboarding flow (current in code — modal overlay)
- `showOnboardingPage()` → modal overlay over Discover
- Step 1: Resume upload
- Step 2: Search intent (title, location, industries)
- Step 3: Confirmation
- `obFinish()` → `showDiscover()`

### Onboarding flow (V4 target — standalone pages)
- 6 distinct view states replacing modal overlay
- See ob_step1 through ob_step6 mocks

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

## Known Bugs (logged, not yet fixed)

| Bug | Severity | Notes |
|---|---|---|
| Paywall race condition on fresh signup | Medium | `authBootGate` paywall check sometimes shows old onboarding modal instead of `ob_step6` on first load immediately after signup. Resolves on hard refresh. Root cause not yet diagnosed — suspect timing between Supabase `users` row creation and the paywall query. Logged June 29, reproduced on `rg9` and `rg10` test signups. |
| `rgInitAnalytics` ReferenceError | Low | `Uncaught ReferenceError: supabase is not defined` in `rgTrack` at L2301, called from `rgInitAnalytics`. Pre-existing, not introduced by Session A. One-line guard fix needed (`typeof supabase !== 'undefined'` check before call). |
| Stripe Link button on ob_step6 | Low | Stripe.js auto-injects a "Save with Link" button in the card element. Not part of design. Needs `disableLink: true` in Stripe Elements options. Cosmetic only, not blocking. |

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

