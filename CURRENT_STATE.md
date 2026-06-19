# Role Garden — CURRENT STATE

> **Last updated:** Day 26 — Friday June 19, 2026 (A.8.6 + auth fix + card fixes + onboarding flow)
> **Purpose:** Architectural reality for build chats. Not a changelog. Not a roadmap.

---

## 1. Product framing

Role Garden is a premium AI career growth platform for professionals seeking their next role at exceptional companies — across industries, across functions. Single tier $39/mo with 14-day card-on-signup trial. Two primary user modes:

1. **Find Jobs** — Matches experience (search + recommendations + Score This Job)
2. **Work a Job** — Applications experience (Assessment + prep workflows + AI copilot)

Everything else supports one of those two modes.

---

## 2. Tech stack

| Component | Detail |
|---|---|
| **Frontend** | Single-file vanilla JS HTML SPA at `app.rolegarden.com` (~14,000+ lines, ~800 KB) |
| **Hosting** | Netlify — auto-deploys from GitHub `martinsmazza-eng/deploy_rg` |
| **Proxy** | Render at `rg-proxy-flpd.onrender.com` (Claude API calls, send-email) |
| **DNS** | Cloudflare (apex + send.rolegarden.com) |
| **Database** | Supabase Pro — `https://xveicdwpxwbezwncbxbp.supabase.co` |
| **Email** | Resend via `send.rolegarden.com` |
| **LLM** | Anthropic API Tier 2. Haiku for ~90% of features. Sonnet for M1D + Frameworkless Prep only |
| **Marketing site** | `rolegarden.com` — Netlify, repo `martinsmazza-eng/rolegarden-marketing`. Holding page with noindex. Framer site Phase C. |
| **Auth** | Email/password via Supabase. OAuth (Google + Microsoft) = Phase B work |

**Local working directories:**
- `~/Documents/deploy_rg/` — frontend repo
- `~/rolegarden_seed/` — seed repo

**GitHub repos:**
- `martinsmazza-eng/deploy_rg` — frontend
- `martinsmazza-eng/rolegarden_seed` — seed
- `martinsmazza-eng/rg-proxy` — Render proxy
- `martinsmazza-eng/rolegarden-marketing` — marketing homepage (rolegarden.com)

**Domains:**
- `rolegarden.com` — marketing homepage (Netlify, noindex holding page)
- `app.rolegarden.com` — app (Netlify)
- `rolegarden.com` — marketing site (Framer, Phase C work)
- `send.rolegarden.com` — Resend transactional
- `rg-proxy-flpd.onrender.com` — Render proxy

---

## 3. Search engine architecture

### 3.1. Pool sourcing — 100% ATS-direct

Three adapters fetch jobs from public APIs. JSearch removed permanently (Day 23).

| Provider | API endpoint | Companies tagged | Notes |
|---|---|---|---|
| **Greenhouse** | `boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true` | 63 (40 high + 22 medium + 1 low) | Stripe, Datadog, MongoDB, Anthropic, The Trade Desk, HubSpot, etc. Single GET, all jobs in one response |
| **Lever** | `api.lever.co/v0/postings/{slug}?mode=json` | 12 (4 high + 8 medium) | Klaviyo, Outreach, Aircall, Tinuiti, ContentSquare. Returns array directly (no wrapper) |
| **Ashby** | `api.ashbyhq.com/posting-api/job-board/{slug}?includeCompensation=true` | 19 (all high) | OpenAI, Notion, Plaid, Ramp, Vanta, Cohere, Sentry, Demandbase, + 11 more. Returns `{ "jobs": [...] }` |

**Total: 94 ATS-tagged curated companies.** Remaining 230+ curated companies (Workday-hosted, custom systems, agencies) are SKIPPED in seed — logged as `curated_skipped`, no fallback.

### 3.2. Seed pipeline (`seed_jobs.js` — Day 25 A.8.5 patch, 1,602 lines)

1. Load curated companies from `rg_curated_companies` where `ats_provider IS NOT NULL`
2. JS slug dedup — prevents multi-bucket companies from being processed twice.
3. For each company → adapter fetches jobs
4. US-only location filter at adapter level — all three adapters. Non-US remote exclusion tightened: "Remote - Ireland", "Remote Canada" etc. now excluded.
5. Title pre-filter — 28 stable universal exclusions. Engineering / PM / design / data / sales / marketing all KEPT.
6. Each job → Haiku classification → 25-bucket industry assignment + `role_category` title fallback + confidence score
7. Drop if: `reject_other` OR `role_match_no` OR `confidence < 0.6` OR `classification_failed`
8. `assignTitleBucket(title, company)` → 15-bucket regex (fast, zero cost). If null → falls back to Haiku `role_category` (already classified in step 6, zero extra cost).
9. Insert with `priority_boost = true` if classified bucket matches curated bucket
10. `expires_at = now() + 60 days` on every insert
11. Dedup on `ats_canonical_id` (ATS source wins on collision)
12. Final log: `rg_seed_run_<timestamp>.json`

**Last full seed (Day 23):** 1,478 inserted (1,375 unique), 91% priority_boost=true, 38 min, $5.30 Haiku cost. Note: Ashby adapter was broken during this run — 0 Ashby jobs inserted. Fixed Day 24 A.2.

### 3.3. Database schema (key tables)

**`rg_curated_companies`** (~327 rows):
- `company`, `industry_bucket` — composite PK (same company can exist in multiple buckets)
- `priority`, `active`
- `ats_provider`, `ats_slug`, `ats_confidence` (high / medium / low)
- `website`, `logo_url`, `logo_scraped_at`
- `greenhouse_url`, `lever_url`, `ashby_url`, `workday_url`
- `crawl_status`, `last_verified_at`, `last_successful_crawl_at`, `last_error`
- `tenant`, `site_id`, `cluster` (Workday metadata — populated when Workday adapter ships in V2)
- `source_url`, `source_type`, `discovery_method`, `notes`

**`rg_jobs_index`** (~1,478 rows, pre-Ashby-fix):
- `job_id`, `title`, `company`, `location`, `remote`
- `description`, `salary`, `apply_url`, `logo_url`
- `posted_at`, `fetched_at`, `expires_at`
- `title_bucket` — 15 buckets via regex; null falls back to Haiku `role_category` (zero extra cost). Jobs with null title_bucket still visible via industry_buckets.
- `industry_buckets` (text[] — up to 2 classified buckets)
- `priority_boost` (boolean)
- `classification_confidence` (0-1)
- `source`, `apply_source`, `source_type` (ats), `ats_canonical_id`, `ats_provider`

**`rg_seed_logs`** — Pattern B logging. JSON snapshots per run.

### 3.4. Frontend search flow

1. User on `/matches` triggers `runM0Search` (auto on load or chip change)
2. `fetchFromIndex()` reads user buckets from profile cache (`rg_career_profile_cache.assigned_title_buckets` + `assigned_industry_buckets`). Extraction fires on all upload paths (onboarding + banner + Profile page) as of Day 24 A.5.5. Falls back to chip-derived buckets if cache empty.
3. `expandUserBuckets()` expands via mega-cluster map (see 3.5)
4. Supabase query: `rg_jobs_index` with `title_bucket IN [...]` + `industry_buckets && [...]` + location + remote filters
5. Returns up to 100 jobs (Day 24 A.7: raise to 300 + cascade scoring)
6. `scoreJobs()` calls Haiku per job in parallel batches → score + dimensions + summary
7. `priority_boost` field preserved on job objects — score inflation (+10) removed Day 24 A.5. Field needed for A.7 pre-rank.
7.5. `getBucketMatchTier()` assigns each job a tier (2/1/0) based on profile primary bucket overlap. `renderJobResults` sorts by `[bucketMatchTier DESC, score DESC]` — Tier 2 = exact primary bucket match, Tier 1 = cluster-adjacent, Tier 0 = out-of-cluster.
8. Cards render via `rgLogoHtml()` priority chain (see 3.6)
8.5. **Cache warm on return visit** — `rg_last_search_cache` saves `preRanked` (full unscored pre-ranked pool) + `cascadeOffset` alongside `results`. On return visit, `rgLoadSearchCache` restores cascade state so Load More works immediately without a fresh Supabase fetch. Pre-scores next batch in background on restore. Fresh cache only — stale cache fires `runM0Search()` as before.

### 3.5. Industry bucket mega-cluster map

```javascript
const INDUSTRY_BUCKET_CLUSTERS = {
  // GTM + Media mega-cluster (7 buckets pool together)
  'advertising_tech':       ['advertising_tech','fintech','retail_ecommerce_tech','sales_saas','marketing_tech','marketing_advertising','media_entertainment'],
  'fintech':                ['advertising_tech','fintech','retail_ecommerce_tech','sales_saas','marketing_tech','marketing_advertising','media_entertainment'],
  'retail_ecommerce_tech':  ['advertising_tech','fintech','retail_ecommerce_tech','sales_saas','marketing_tech','marketing_advertising','media_entertainment'],
  'sales_saas':             ['advertising_tech','fintech','retail_ecommerce_tech','sales_saas','marketing_tech','marketing_advertising','media_entertainment'],
  'marketing_tech':         ['advertising_tech','fintech','retail_ecommerce_tech','sales_saas','marketing_tech','marketing_advertising','media_entertainment'],
  'marketing_advertising':  ['marketing_advertising','media_entertainment','advertising_tech','marketing_tech','sales_saas'],
  'media_entertainment':    ['media_entertainment','marketing_advertising','advertising_tech','marketing_tech','sales_saas'],

  // Infrastructure cluster
  'dev_tools':              ['dev_tools','cybersecurity','ai_ml_companies','sales_saas'],
  'cybersecurity':          ['cybersecurity','dev_tools','ai_ml_companies','sales_saas'],
  'ai_ml_companies':        ['ai_ml_companies','dev_tools','cybersecurity','sales_saas'],

  // Domain-specific
  'healthtech':             ['healthtech','sales_saas'],
  'edtech':                 ['edtech','sales_saas','marketing_tech'],
};
```

Frontend logs `[fetchFromIndex] Cluster expansion: X primary → Y cluster buckets` on every search.

**Note:** 12 new industry buckets exist in the seed classifier (legaltech, hrtech, proptech, climate_clean_energy, logistics, insurtech, govtech, gaming, consumer_apps, hospitality_travel, food_beverage_tech, manufacturing_tech) but are NOT yet in `INDUSTRY_BUCKET_CLUSTERS`. Frontend search won't expand these new verticals until added. Phase B work. `[FOLLOW-UP]`

### 3.6. Logo render priority chain

```
1. Supabase logo_url (og:image scraped at seed time) — populated for ~236 companies
2. Logo.dev: https://img.logo.dev/{domain}?token=pk_aQ3DjrEPSiGnOWG0MBV1BQ&size=200&format=png
3. Brandfetch onerror fallback — legacy, kept as safety net
4. Initials avatar — final fallback (colored circle, first letter)
```

Logo.dev token domain-restricted to `app.rolegarden.com` (server-side enforced). Token in JS source is fine.

### 3.7. Scoring (current implementation)

`scoreJobs()` — Haiku call with 4-dim rubric:
- **Industry fit** (0-55 pts)
- **Role fit** (0-25 pts)
- **Seniority fit** (0-15 pts)
- **Skills fit** (0-10 pts, only if industry passed)

Total: 0-100. Priority_boost score inflation removed Day 24 A.5. Sort: `[bucketMatchTier DESC, score DESC]` — Tier 2 = exact primary bucket match, Tier 1 = cluster-adjacent, Tier 0 = out-of-cluster.

---

## 4. What's broken (active P0/P1 list)

| # | Issue | Severity | Status |
|---|---|---|---|
| 4.1 | Ashby adapter: `data.jobPostings` → `data.jobs` key mismatch | P0 | **FIXED Day 24 A.2** (commit 213b61a). Mini-seed verified Ramp + OpenAI. Full seed pending. |
| 4.2 | Haiku classifier is sales-only — drops engineering/PM/design as `role_match_no` | P0 | **FIXED Day 24 A.3** — prompt rewrite, all knowledge-worker roles now pass. `role_match_no` reserved for warehouse/food service/manual labor only. Full seed pending. |
| 4.3 | Sort by raw score, not bucket-match-tier | P0 | **FIXED Day 24 A.5** — `getBucketMatchTier()` + `[tier DESC, score DESC]` sort in `renderJobResults` |
| 4.6 | Onboarding skip + resume-via-banner path doesn't fire profile extraction | P1 | **FIXED Day 24 A.5.5** — `extractCareerProfile()` added to `saveResumeText()`. Fires on all upload paths. |
| 4.7 | Left rail title/industry chips conflict with Phase B "My Profile" UX | P1 | **FIXED Day 24 A.5.5** — TITLES and INDUSTRIES hidden (`display:none`). DOM intact for Phase B re-show. |
| 4.4 | International remote leakage — Greenhouse adapter has no US-only filter | P1 | **FIXED Day 24 A.4** — `isGreenhouseUSJob()` added, conservative logic, all three adapters now have US filter |
| 4.5 | Dupe processing — Stripe/HubSpot/Klaviyo have 2 curated rows, fetched twice | P1 | **FIXED Day 24 A.6** — JS slug dedup in `main()` after curated companies load |

| 4.8 | Seed header log shows stale JSearch references ("150 without ATS will use JSearch fallback", "Title buckets: 2 (6 variants)") | P1 | Open — fix in next full re-seed session `[FOLLOW-UP]` |
| 4.9 | `priority_boost` still used as pre-rank signal in `preRankJobs()` — JSearch gone, field adds no differentiation. Remove from tier logic, use title+industry match only. | P1 | Open — 2-line fix in index.html `preRankJobs()` `[FOLLOW-UP]` |
| 4.10 | `INDUSTRY_BUCKET_CLUSTERS` missing 12 new verticals — frontend search won't expand new vertical buckets | P1 | Open — Phase B work `[FOLLOW-UP]` |
| 4.11 | Auth session not established when `runM0Search()` fires after onboarding → 401 → no results on first load | P0 | **FIXED Day 26** — onAuthStateChange listener gates search on SIGNED_IN/INITIAL_SESSION. `_rgAuthSearchFired` one-shot guard prevents double-fire. |
| 4.12 | Cascade render mixing scored + unscored cards — bottom cards show without Why/Challenge/summary content | P0 | **FIXED Day 26** — validScored filter in cascadeLoadMore + cache restore guard. renderSkeletonCards() detach/re-attach also fixed. |
| 4.13 | Load More button disappearing after Search button click | P1 | **FIXED Day 26** — root cause: renderSkeletonCards() destroying rg-load-more-container. Fixed with detach/re-attach pattern. Label changed to "Load More". |
| 4.14 | Background search during onboarding — user lands on empty state | P1 | **FIXED Day 26** — obFinish() now awaits extraction first (series not parallel), then fires fetchFromIndex + preRankJobs + scoreJobs(batch1). Cards ready on landing. |
| 4.15 | rolegarden.com noindex — remove before paid media launch | P1 | Open `[FOLLOW-UP]` |
| 4.16 | HTML-encoded job descriptions sent to Haiku — score=50 fallback, no summary/why/gaps | P0 | **FIXED Day 26** — stripHtml() regex-based (decode entities + strip tags). slice raised 200→1500 chars. |
| 4.17 | scoreJobs batch hard-capped at 6 jobs — jobs 7-10 always score=50 | P0 | **FIXED Day 26** — jobs.slice(0,6) → jobs.slice(0,10). Root cause of all prior max_tokens escalations (1000→8000). |
| 4.18 | jobIdx=-1 bug — Create Opportunity + Save for Later silently fail | P0 | **FIXED Day 26** — renderJobResults uses findIndex by job.id + pushes cascade jobs into m0Results. |
| 4.19 | Skip reason picker showing on every skip | P1 | **FIXED Day 26** — skipJobByIdx() simplified: skips immediately, no picker. |
| 4.20 | saveSkipped() not defined — skip throws ReferenceError | P0 | **FIXED Day 26** — added saveSkipped() next to m0SkippedJobs declaration. |
| 4.21 | M1 assessment auto-runs on opportunity creation | P1 | **FIXED Day 26** — removed setTimeout(runM1, 800) from addJobAsOpportunity(). Assessment now user-initiated only. |
| 4.22 | Opportunity jobs not suppressed from future searches | P1 | **FIXED Day 26** — addJobAsOpportunity() pushes to m0SkippedJobs with reason='opportunity_created'. |
| 4.23 | M1A workspace uses 5-dim scoring vs search card 4-dim — dimensions mismatch | P1 | Open — Phase B (B.9 roadmap item) `[FOLLOW-UP]` |
| 4.24 | Dream Companies autocomplete showing email — regressed | P1 | Open `[FOLLOW-UP]` |

### 5.1. Screens and wiring

- **Login / signup** — email/password, Supabase auth
- **Onboarding** — 3 steps: Resume upload → Career Profile (Haiku-extracted) → Ready
- **Matches page** — pre-redesign layout:
  - Left rail: LOCATION + REMOTE toggle + DREAM COMPANIES (TITLES and INDUSTRIES hidden Day 24 A.5.5 — Phase B re-expose via My Profile widget)
  - Center: Match cards (score ring + summary + actions)
  - Right rail: not yet built (Phase B)
- **Applications page:**
  - Left rail: Application list
  - Center: M1A Assessment + 4-dim scoring + Strengths/Gaps
  - Workspace tabs: Match Assessment (M1A) | Recruiter Prep (M1B) | HM Prep (M1C) | Final Round (M1D)
- **Saved Jobs** — saved card list
- **Saved Searches** — renamed from Saved Matches
- **Dream Companies** — tracked companies list (moves to Profile widget in Phase B redesign)
- **Pricing page, Settings, ToS** — placeholder state

### 5.2. M1A/B/C/D status

| Module | Status | Notes |
|---|---|---|
| **M1A Match Assessment** | SHIPPED | 4-dim scoring. Deeper Match Analysis (3-4 paragraph reasoning) = Phase B work |
| **M1B Recruiter Prep** | SHIPPED | 6 sections. STAR mapping in M1A only (M1B Section 7 removed Day 7.5) |
| **M1C HM Prep** | SHIPPED | Carry-forward from M1B user messages |
| **M1D Final Round / Strategy Coach** | NOT SHIPPED | Text-only, NO deck export in V1 (locked June 5). Sonnet-powered. Phase C work (~6-9h) |
| **Frameworkless Prep** | SHIPPED | Open-ended grounded chat. Sonnet-powered. Session 3.6 |

### 5.3. Resume engine status

| Component | Status |
|---|---|
| Resume parser (text extraction PDF/DOC/DOCX/TXT via mammoth.js) | SHIPPED |
| Structured extraction (`parseResumeStructured`) | NOT SHIPPED — Phase C |
| Tailoring engine | NOT SHIPPED — Phase C |
| LightningPDF integration | NOT SHIPPED — Phase C |

---

## 6. Locked decisions (do not relitigate)

- **$39/mo single tier**, 14-day card-on-signup trial, full feature parity trial + paid
- **Auto-search enabled** in trial + paid (no gating)
- **Haiku for ~90% of features** — Sonnet only for M1D + Frameworkless Prep
- **Role Garden Lite $9/mo** — cancellation-save offer only, NOT publicly marketed
- **JSearch removed permanently** — ATS-direct only (Day 23 lock)
- **V1/V2 framing only** — no alpha/beta/V1.5. Audience expansion phases are within V1
- **M1D V1 scope** — text-only Strategy Coach, NO deck export (locked June 5)
- **Three-column Matches layout** — per `ROLE_GARDEN_V2_LAYOUT_LOCK.md` (Saved Searches rail + Recommendations + Score This Job / Profile)
- **Saved Matches → Saved Searches** (renamed)
- **Match labels:** Excellent / Strong / Good / Fair (no numeric score shown)
- **Dream Companies** moves into Profile widget (not left rail) in Phase B
- **Bundle migration** (single HTML → React/Vite) — V2 work, not V1
- **Marketing positioning:** "premium job search platform" + "curated companies across industries" + "ATS-direct, no scraping" trust differentiator. NOT a sales job board — targets all professionals (engineers, PMs, designers, consultants, ops, finance, legal, data, marketing) seeking roles at exceptional companies.
- **Prompt security:** Haiku scoring + card generation prompts currently visible in browser DevTools (index.html deployed to Netlify). Moving to Render proxy pre-paid-media (B.3). Company registry + ATS slugs already protected in Supabase. Seed classifier prompt in private repo only. — target ~$5-10/month seed cost. Full TRUNCATE + re-seed only for major schema changes. Baked into Phase B Edge Function migration.
- **Scoring cost lever:** cascade scoring only (A.7). No description trimming (card quality depends on full JD). No pre-scoring (profile distribution unknown). Score caching = Phase B candidate after real usage data.

---

## 7. Source-of-truth document priority

For any scope or architecture question, read in this order:

1. **`CURRENT_STATE.md`** (this doc) — architectural reality
2. **`ROLE_GARDEN_V2_LAYOUT_LOCK.md`** — Matches design canonical (merged + locked)
3. **`_ROADMAP.md`** — sprint state, session history
4. **`driver_chat_operating_manual.md`** — process discipline
5. **`_CONSTITUTION.md`** — non-negotiable rules
6. **`BRIEF_TEMPLATE.md`** — build chat brief structure
7. **`pricing_model_locked.md`** — pricing decisions
8. **PRDs in `features/`** — per-feature specs
