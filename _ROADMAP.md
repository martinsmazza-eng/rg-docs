# Role Garden — Sprint Roadmap

> **Last updated:** Day 24 — Wednesday June 17, 2026 (Session A.2 shipped)
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
| **seed_jobs.js** | Day 24 A.2 patch (ATS-direct only, Ashby fixed) |
| **Pool state** | ~1,478 jobs (pre-Ashby-fix). Full re-seed pending A.3 + A.4 completion |
| **Cost per seed** | ~$5.30 current → target ~$2 after A.3 multi-role classifier fix |

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
**Status: OPEN — next up**
- Current classifier prompt is sales-only — drops engineering/PM/design as `role_match_no`
- Rewrite: accept multiple role categories per industry bucket (sales, engineering, product, design, data, marketing)
- Expected: ~40% Haiku cost cut + multi-role pool coverage
- Brief: `RG_D24_A3_multi_role_classifier_brief.md`

### Session A.4 — Greenhouse US-only location filter
**Status: OPEN**
- Lever + Ashby already have US-only filter (Day 24 A.2)
- Greenhouse adapter missing — international remote leakage (Datadog Chile, Asana Switzerland, etc.)
- Fix: US filter in GreenhouseAdapter `parseLocation()` matching Lever/Ashby pattern
- Brief: `RG_D24_A4_greenhouse_us_filter_brief.md`

### Session A.5 — Sort by bucket-match-tier
**Status: OPEN**
- Current sort: `score DESC` — premium brand cards mixed with noise
- Add `bucketMatchTier`: Tier 2 = exact primary bucket, Tier 1 = cluster-adjacent, Tier 0 = out-of-cluster
- Sort: `[bucketMatchTier DESC, score DESC]`
- Render tier groups with subtle visual separator
- Brief: `RG_D25_A5_sort_by_bucket_tier_brief.md`

### Session A.6 — Dupe DISTINCT fix
**Status: OPEN**
- Stripe / HubSpot / Klaviyo have 2 curated rows each (multi-bucket) — seed fetches them twice
- Fix: `DISTINCT ON (ats_provider, ats_slug)` in curated company query
- Brief: `RG_D25_A6_dupe_distinct_brief.md`

### Session A.7 — Cascade scoring + 300 cap + Load More
**Status: OPEN**
- Raise pool fetch from 100 → 300 jobs
- Score first 10 immediately, render
- "Load 10 More" button → score next batch on click
- In-memory cascade state (no localStorage)
- Expected: ~75% Haiku cost cut on initial search
- Brief: `RG_D25_A7_cascade_scoring_brief.md`

### Session A.8 — TRUNCATE + full re-seed + smoke test
**Status: OPEN**
- Verify all Phase A patches applied
- TRUNCATE `rg_jobs_index`
- Full seed run (expect ~10-15 min, ~2,800 jobs, ~$2 cost)
- Browser smoke test incognito
- Pattern B analysis on new log
- **Phase A close target:** ~2,500-3,500 jobs, multi-role, cleanly sorted, ~$2/seed, all P0 bugs fixed

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
Move `seed_jobs.js` from local Node → Supabase Edge Function. pg_cron nightly 2am ET. Removes Mac-must-be-on dependency. Email digest via Resend on completion.

### B.6 — Custom system adapters: Netflix / Google / Meta (Day 30)
Maintained scrapers for big-brand career pages not on ATS public APIs. ~50-150 jobs/refresh. Weekly schedule.

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

### C.9 — Industry/matrix expansion (Day 38)
12 more buckets: legaltech, hrtech, proptech, climate_clean_energy, logistics, insurtech, govtech, gaming, consumer_apps, hospitality_travel, food_beverage_tech, manufacturing_tech. Title buckets beyond 2: revenue_operations, customer_success, partnerships, business_development.

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
| 24 | June 17 | **Session A.1:** workflow + doc management overhauled, rg-docs repo created. **Session A.2:** Ashby adapter fixed (jobPostings→jobs, dept filter removed, US filter added). Commit 213b61a. |
