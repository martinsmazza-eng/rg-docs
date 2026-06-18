# Role Garden — Sprint Roadmap

> **Last updated:** Day 25 — Thursday June 18, 2026 (A.8.5 shipped)
> **Purpose:** Sprint state + session-by-session plan through V1 paid-media launch (7/6/2026).
> **Framing lock:** V1 (current build, expanding audience) + V2 (next major) only. No alpha/beta/V1.5.

---

## Sprint header

| Field | Value |
|---|---|
| **Current phase** | V1 — Phase A Foundation Refinement |
| **Day 24** | Wednesday June 17, 2026 |
| **Paid media target** | Sunday July 6, 2026 (Day 40) |
| **Pace** | ~5-7h/day solo |
| **index.html** | ~14,000+ lines, ~800 KB |
| **seed_jobs.js** | Day 25 A.8.5 patch (1,602 lines — Haiku title fallback, 25 buckets, remote fix, non-US remote exclusion) |
| **Pool state** | ~1,478 jobs (pre-full-reseed). Full re-seed pending A.4 completion |
| **Cost per seed** | ~$5.30 current → target ~$2 after full re-seed with A.2+A.3 fixes |

---

## Phase overview

| Phase | Days | Theme | Est. effort |
|---|---|---|---|
| **A** | 24-25 | Foundation refinement | ~7-9h |
| **B** | 26-32 | Experience overhaul | ~40-55h |
| **C** | 33-39 | Launch readiness | ~30-45h |
| **Launch** | 40 (Sun 7/6) | Paid media on | — |

---

## PHASE A — Foundation refinement

**Days 24-25 (Wed-Thu June 17-18). Goal: fix what's broken in search, unblock dev flow.**

### Session A.1 — Workflow documentation cleanup
**Status: SHIPPED Day 24**
- Workflow doc updated, DRIVER_CHAT_OPENING aligned, brief template finalized
- rg-docs GitHub repo created (CURRENT_STATE + ROADMAP source of truth)
- Build chat opening prompt format standardized (chat name + copyable block)
- Post-session doc update workflow locked (driver chat owns roadmap/current state, build chat proposes session-scoped changes only)

### Session A.2 — Ashby adapter bug fix
**Status: SHIPPED Day 24** (commit `213b61a`)
- Root cause: `data.jobPostings` → `data.jobs` key mismatch (API returns `jobs`, adapter read `jobPostings`)
- Redundant department filter removed (TITLE_PREFILTER_REGEX + Haiku handle role filtering)
- US-only location filter added to AshbyAdapter (matches Lever adapter pattern)
- Null-guard added: warns with slug + actual keys if API shape changes
- Field mapping audit: all fields already correct, no changes needed
- Sentry / Demandbase / Ramp confirmed Ashby-only (no Greenhouse conflict)
- Mini-seed verified: Ramp + OpenAI both returned `raw_jobs > 0`, inserts confirmed
- **~1,400 Ashby jobs recoverable on next full seed**

### Session A.3 — Multi-role Haiku classifier rewrite
**Status: SHIPPED Day 24**
- Root cause: prompt explicitly dropped all non-sales roles as `role_match_no` + `reject_other`
- Prompt rewritten: classifier now handles industry fit only — all knowledge-worker roles pass (sales, engineering, PM, design, data, marketing, ops, finance, legal)
- `role_match_no` now reserved for non-knowledge-worker roles only (warehouse, food service, clinical frontline, manual labor)
- `role_match` schema simplified: yes/adjacent/no → yes/no (`adjacent` was legacy artifact)
- `role_match` fallback changed: `|| 'no'` → `|| 'yes'` (parse failures no longer silently drop jobs)
- Static checks passed. Mini-seed verified in build environment. Full seed pending A.4.

### Session A.4+A.6 — Title bucket expansion + Greenhouse US filter + Dedup fix
**Status: SHIPPED Day 24** (1,464 → 1,539 lines)
- Greenhouse US-only filter added (`isGreenhouseUSJob()` — conservative string matching, all three adapters now covered)
- `assignTitleBucket(title, company)` — 15-bucket regex function replaces hardcoded `account_executive` in all 3 adapter `normalizeToJobSchema()` calls
- "Lead, Incidents & Escalations, User Operations" (OpenAI) root cause fixed — returns `null`, not `account_executive`
- 15 title buckets: account_executive, sales_director, business_development, revenue_operations, customer_success, product_manager, software_engineer, data, design, marketing, operations, consulting, finance, legal, people_hr
- `expires_at` corrected: `posted_at + 14d` → `now() + 60d` on insert across all adapters
- JS slug dedup in `main()` — Stripe/HubSpot/Klaviyo now processed once per seed run
- `priority_boost` +10 score inflation confirmed in `index.html` only — deferred to A.5
- Backup: `~/Documents/RG_backups/seed_jobs_pre_A4A6_20260617_190342.js`

### Session A.6 — Dupe DISTINCT fix
**Status: SHIPPED Day 24** — bundled into A.4+A.6 above.

### Session A.5 — Sort by bucket-match-tier + priority_boost cleanup
**Status: SHIPPED Day 24** (commit `8ea14ca`, +38/-29 lines in index.html)
- `getBucketMatchTier(job, userPrimaryBuckets)` — Tier 2 = exact primary bucket match, Tier 1 = cluster-adjacent, Tier 0 = out-of-cluster
- `renderJobResults` sort replaced with `[bucketMatchTier DESC, score DESC]`
- `priority_boost` +10 score inflation removed — JSearch gone, all jobs ATS-direct
- `priority_boost` field preserved on job objects for A.7 pre-rank
- `userPrimaryBuckets` resolved from `localStorage.rg_career_profile_cache` at sort time — no signature changes
- Smoke test confirmed tier sort firing. Tier 2: 0 due to profile extraction bug (see A.5.5)

### Session A.5.5 — Profile extraction fix + hide left rail chips
**Status: SHIPPED Day 24**
- Root cause: `saveResumeText()` didn't call `extractCareerProfile()` — only `obFinish()` did. Banner/Profile upload path left users with no buckets → Tier 2 always 0.
- Fix: `extractCareerProfile()` added to `saveResumeText()` — fires on all upload paths. On success writes buckets to cache, triggers `runM0Search()` with 300ms delay.
- TITLES and INDUSTRIES hidden from left rail (`display:none`). DOM intact for Phase B re-show.
- LOCATION, REMOTE toggle, DREAM COMPANIES unchanged and visible.
- Closes P1 bugs 4.6 + 4.7. 15,581 → 15,613 lines.

### Session A.7.5 — Load More CSS + Dream Companies guard + Cache warm
**Status: SHIPPED Day 25** (15,738 → 15,787 lines, +49)
- Load More button moved inside `#m0Out` — was flex-sibling of m0Out in `.discover-shell`, rendered top-right as third column
- Boot-time email guard: filters `@` from `rg_user_intent.target_companies` on every startup — clears legacy-contaminated localStorage
- Cache warm: `rg_last_search_cache` now saves `preRanked` pool + `cascadeOffset` — cascade resumes on return visit without re-fetch, pre-scores next batch in background
**Status: OPEN**
- **Pre-rank (Stage 1, client-side, 0 cost):** Sort 300 jobs before Haiku sees them using lightweight signals: (1) exact title_bucket + exact industry + priority_boost, (2) exact title_bucket + exact industry, (3) exact title_bucket + cluster industry + priority_boost, (4) exact title_bucket + cluster industry, (5) cluster industry + priority_boost, (6) cluster industry only, (7) everything else. Best-aligned jobs guaranteed to surface in batch 1. Runs in ~5ms, no API call.
- **Remove priority_boost score inflation:** Already removed in A.5. Pre-rank uses field as ordering signal only.
- **Cascade scoring (Stage 2):** Score first 10 in pre-ranked order immediately. Background pre-score next batch while user reads batch 1. Load More → instant render (already scored). Rolling 1-batch-ahead pre-fetch.
- **Skeleton cards:** Show placeholders on Load More click, fill as scores arrive. Safety net if pre-score not ready.
- **300 cap + stale-while-revalidate:** Fetch 300 from Supabase. No arbitrary cap — pre-rank ensures quality at top. Returning users served cache instantly, fresh fetch runs in background, soft-refresh when ready.
- **Chip changes:** Pre-rank re-runs client-side against same 300 jobs on chip change. No new Supabase query unless location/remote changes.
- Expected: ~75% Haiku cost cut on initial search + near-zero perceived latency
- Brief: `RG_D25_A7_prerank_cascade_scoring_brief.md`

### Session A.8.5 — Haiku title bucket + remote fix + new verticals + non-US remote exclusion
**Status: SHIPPED Day 25** (1,539 → 1,602 lines, +63)
- Haiku `role_category` added to classification prompt — zero extra cost, fills null title_bucket gaps
- Remote field fix: Ashby `isRemote` boolean + "United States"/blank → `remote=true` across all adapters
- 25-bucket classifier: 12 new industry verticals added to prompt + `VALID_BUCKETS`
- Non-US remote exclusion: "Remote - Ireland", "Remote Canada" etc. now correctly excluded
- Mini-seed: Trade Desk ✅, OpenAI ✅ (new verticals appearing in secondary buckets), Deel 0 jobs (no US remote roles posted)
- `[FOLLOW-UP]` Remove `priority_boost` from `preRankJobs()` tier logic — 2-line fix in index.html
- `[FOLLOW-UP]` Add 12 new verticals to `INDUSTRY_BUCKET_CLUSTERS` in index.html — Phase B
- `[FOLLOW-UP]` Strip JSearch references from seed header log

### Session A.8 — TRUNCATE + full re-seed + smoke test
**Status: OPEN**
- Verify all Phase A patches applied
- TRUNCATE `rg_jobs_index`
- Full seed run (expect ~20-30 min, **~3,000-4,000 jobs** across 15 title buckets from 94 companies, ~$3-4 cost)
- Browser smoke test incognito — verify multi-role results surfacing (PM, engineer, designer, AE)
- Pattern B analysis on new log
- Verify `expires_at` populated on all rows
- **Phase A close target:** ~3,000-4,000 jobs, 15 title buckets, multi-role, cleanly pre-ranked + sorted, ~$3/seed, all P0 bugs fixed

---

## PHASE B — Experience overhaul

**Days 26-32 (Fri June 19 → Thu June 25). Goal: visible product upgrades.**

### B.1 — App redesign: 3-col Matches layout (Day 26)
Per `ROLE_GARDEN_V2_LAYOUT_LOCK.md`. Left: Saved Searches rail. Center: Recommended Matches. Right: Score This Job (~66%) + Profile widget (~34%). Dream Companies moves into Profile widget.

### B.2 — Match card visual restructure (Day 27)
"Why You'll Love It" / "Potential Challenge" / "Fit Summary" / "Recommendation". Labels: Excellent / Strong / Good / Fair. No numeric score. No red states.

### B.3 — Score This Job widget (Day 28)
Right-rail primary widget. Single intelligent input (JD paste / URL / recruiter email). Scored job renders as highlighted card at top of results.

### B.4 — Frontend design polish: typography + spacing (Day 28)
Premium spacing, Manrope tuning, card hover states + transitions.

### B.5 — Phase 2b Edge Function migration (Day 29)
Move `seed_jobs.js` from local Node → Supabase Edge Function. pg_cron 3x/week schedule (not nightly). Removes Mac-must-be-on dependency. Email digest via Resend on completion. **Implement incremental seeding here:** filter by `publishedAt > now() - 48h` before classifying — skip jobs already in DB by canonical ID. Target: ~$5-10/month seed cost vs ~$90/month nightly full re-seed. Full TRUNCATE + re-seed reserved for major schema changes only.

### B.6 — Custom system adapters: Google + Meta (Day 25 post-Phase-A), Amazon (Day 30), Workday spike (Day 26)
- **Google** (`careers.google.com`) — undocumented JSON API, stable for years. ~3-4h. Pre-V1.
- **Meta** (`metacareers.com`) — similar undocumented API, reasonably stable. ~3-4h. Pre-V1.
- **Amazon** (`amazon.jobs`) — paginated, rate-limited, more complex. ~6-8h. Phase B Day 30.
- **Workday spike** (EY only, timebox 4h) — Workday public job board XHR endpoint. If pattern works, add 2-3 more consulting firms (KPMG, Deloitte). If fragile, punt to V2. High value (consulting = large user segment). Day 26.
- Google + Meta alone add ~2,000-3,000 premium jobs from two of the most recognizable employers globally.

### B.7 — OAuth: Google + Microsoft (Day 30)
Supabase Auth providers. Update signup + login flows.

### B.8 — M1A Deeper Match Analysis (Day 31)
New Haiku call on opp creation: resume + JD + career profile + score breakdown → 3-4 paragraphs employer-specific reasoning. ~$0.003/opp.

### B.9 — M1A 5-dim → 4-dim migration (Day 31)
M1A workspace `runM1` still bundles legacy 5-dim scoring. Migrate to 4-dim (Industry/Role/Seniority/Skills) for consistency with M0. Touch points: L2431, L3223.

### B.10 — Admin panel + Observability stack (Day 32)
PostHog event capture (app_loaded, search_completed, card_clicked, m1a_opened, opp_created, m1b/c/d_run). Sentry error tracking. Admin route `/admin?key=...` with manual seed trigger + recent seed runs view.

**Phase B close target:** Visible redesign live, nightly seeds automated, multi-role pool, OAuth, deeper M1A. Pool ~3,000-4,500 jobs.

---

## PHASE C — Launch readiness

**Days 33-39 (Fri June 26 → Thu July 2). Goal: monetization + legal + polish.**

### C.1 — Resume engine: structured extraction (Day 33)
`parseResumeStructured(rawText)` → Haiku → JSON {first_name, last_name, email, summary, rest_of_resume}.

### C.2 — Resume engine: tailoring + PDF (Day 33)
`tailorResumeSummary(baseResume, oppJD)` → Sonnet → tailored summary. HTML template → LightningPDF → email-attachable PDF. One ATS-friendly template.

### C.3 — M1D Strategy Coach text-only (Day 34)
Sonnet-generated. Accordion structure. Sections: 5-min pitch, proof points, anticipated objections, questions to ask, the close. NO deck export in V1 (locked June 5).

### C.4 — STAR library workflow revisit (Day 35)
Per-section accordions, Skills section addition.

### C.5 — Mobile browser experience (Day 35)
Search rail collapse, card stack, workspace tabs as horizontal scroll. No swipe-to-save (V2).

### C.6 — Stripe integration + pricing revisit (Day 36)
Card-on-signup flow. Trial countdown UI. Webhook handlers (trial_will_end, payment_succeeded, payment_failed, subscription_canceled). Pricing revisit: re-evaluate $39/mo given ATS-direct positioning.

### C.7 — Transactional emails via Resend (Day 37)
Welcome, trial day 7 reminder, trial day 13 warning, trial expired, payment success/failure, subscription canceled.

### C.8 — Marketing site (Framer) (Day 37)
Landing page for paid traffic. Hero, feature sections, pricing card. ToS + Privacy Policy pages.

### C.9 — Industry expansion (Day 38)
12 new industry verticals, each with curated company list + potential cluster groupings: legaltech, hrtech, proptech, climate_clean_energy, logistics, insurtech, govtech, gaming, consumer_apps, hospitality_travel, food_beverage_tech, manufacturing_tech. Title buckets already expanded to 15 in A.4 — no title work needed here. Focus is company registry expansion per new vertical. Role bucket coverage (revenue_operations, customer_success, partnerships, business_development) already handled in A.4.

### C.10 — ToS lawyer review (Day 38)
Engage attorney with `tos_resume_consent_clause_draft.md`. Start outreach Day 33 (allow ~5-7 days). Est. $1,500-3,000.

### C.11 — Trademark filing — USPTO Class 42 (Day 39)
"Role Garden" trademark, SaaS class. ~$300. 6-12 month registration timeline.

### C.12 — robots.txt + Cloudflare hardening (Day 39)
robots.txt blocks crawlers from `app.rolegarden.com`. Cloudflare in front of Netlify for DDoS + edge caching.

### C.13 — Incident response plan (Day 39)
Security breach playbook, lawsuit/C&D escalation, lights-off procedure. Kill-switch: Render proxy toggle = 30 seconds offline.

**Phase C close:** Stripe paywall live, transactional emails wired, marketing site live, legal hardened, broader industry coverage. **Paid media safe to launch Day 40.**

---

## Day 40 — Sunday July 6 — PAID MEDIA ON

- Meta Ads launch
- Google Ads launch
- LinkedIn Ads launch
- AdCreative.ai ($29/mo) for creative variations
- GA4 + UTM discipline + ChartMogul activation
- Monitor: signups, trial-to-paid, churn
- Daily PostHog dashboard reviews for first 2 weeks

---

## V2 (LOCKED — post-paid-media)

| # | Item | Est. effort |
|---|---|---|
| V2.1 | Employer self-service onboarding | ~10-15h |
| V2.2 | AI agents (RG Ops Agent suite — title filter, logo refresh, seed monitor, slug discovery) | ~15-20h |
| V2.3 | Workday adapter (Disney, Salesforce, Bloomberg, Comcast, JPM) | ~14-18h |
| V2.4 | Feedback tool (floating button + screenshot + Supabase storage + email) | ~3-4h |
| V2.5 | International expansion (UK/EU/Canada) | ~6-8h |
| V2.6 | Bundle migration (single HTML → React/Vite) | ~40-80h |

---

## Shipped milestones (audit trail)

| Day | Date | Shipped |
|---|---|---|
| 8 | May 29 | LLC filed, Google Workspace live, social pages live |
| 11 | June 2 | $39 charm pricing locked, auto-search trial+paid parity |
| 12 | June 3 | Saved searches cap 10, daily auto-search toggle, rail IA |
| 13 | June 4 | scoreJobs Haiku rewrite, 4-dim, career profile, card redesign (Day 10A). M1D V1 = text-only locked |
| 14 | June 5 | Phased launch timeline locked: 6/19 close-friends → 7/6 paid media |
| 17 | June 8 | Day 10A.4 polish, frameworkless prep ships |
| 18 | June 11 | Phase 1 nightly index v5.2 (337 unique rows, 78 queries, 600 inserted) |
| 19 | June 12 | Browser verify Phase 1, GitHub setup, workflow doc plan |
| 20 | June 13 | Phase 2a Haiku classification at seed (v6.0, 673 inserted, 38.5% drop rate) |
| 22 | June 15 | Phase 2c ATS adapter framework (v7.0, Greenhouse+Lever+Ashby, 49 companies tagged, 1,348 inserted) |
| 23 | June 16 | JSearch removed, mega-cluster buckets, Logo.dev chain, priority_boost fix, 92 ATS-tagged companies, 1,478 inserted ATS-only. V1/V2 framing locked. 5 workflow startup docs. |
| 24 | June 17 | **A.1:** workflow + docs. **A.2:** Ashby adapter fixed. **A.3:** multi-role classifier. **A.4+A.6:** 15 title buckets, Greenhouse US filter, slug dedup, expires_at. **A.5:** bucket-tier sort + priority_boost inflation removed. **A.5.5:** profile extraction fix + chip hide. **A.7:** pre-rank + cascade scoring + 300 cap + Load More. 67 new vertical companies + logos. |
| 25 | June 18 | **A.7.5:** Load More CSS fix, Dream Companies email guard, cache warm. **A.7.5 patch:** Load More DOM fix, autocomplete guard. **A.8.5:** Haiku title fallback, 25 buckets, remote fix, non-US remote exclusion. Full re-seed: 8,081 jobs. HubSpot slug fixed. |
