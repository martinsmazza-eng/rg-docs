# _ROADMAP.md — Role Garden Sprint

> **What this is:** Current sprint state, day-by-day plan, what's shipped, what's next. Updated end-of-session.

**Last updated:** May 30, 2026 (Day 9 weekend status. **STRATEGIC DECISION May 29: Path A KILLED.** Marcelo questioned Path A directly given closed-beta mobile fix (Session 5B ships limited mobile = Search + My Opportunities + M1A) removes the original "mobile broken, email is the workaround" need. Both Path A flows replaced: Flow 1 (auto-search delivery) → notification email + deep link (LinkedIn/Indeed pattern, less infra, no inbound parsing, no per-email M1A cost); Flow 2 (user-shared JD) → mobile M1A paste-and-go (strictly fewer steps than forward-and-wait). Removes inbound email parsing infra, `jd@rolegarden.com` subdomain MX, JD-extraction logic, result-email templating. **All Path A references below should be read as historical context, not active V1 plan.** When V1 acquisition is built, the architecture is notification email (via existing Resend outbound on `send.rolegarden.com`) + mobile paste. PRIOR ENTRIES: May 28 documentation sync + two resume-context blockers identified [both since SHIPPED: Session 3.5-R PDF parser May 28 + Session 3.5-M getUserContextForPrompt migration May 28]; Sessions 3.5/3.6/3.7/3.7.1/3.7.2 all shipped May 28-29. Day 8 parallel-track infrastructure SHIPPED May 29: LLC filed on Sunbiz [Role Garden LLC, Northwest RA + Virtual Office bundle, ~$593/yr, 2-5 day approval clock], Google Workspace live [marcelo.m@rolegarden.com + support@/hello@/feedback@ aliases, DKIM authenticated], Cloudflare Email Routing retired with surgical SPF preservation, social pages [FB + LinkedIn + IG] live with consistent brand assets [8 files: circle 512 for FB+Workspace, square 512 for LinkedIn, covers, hi-res]. Day 9 weekend: 3.7.3 fix brief written for follow-up email click bug surfaced in live smoke test [latent 3.7 bug — guard fires silently when notes unsaved due to debounce race; FOLLOW-UP EMAIL duplicate label render bug also caught]; chat-driven follow-up email idea logged to V1 polish per Marcelo's instinct.)

---

## Sprint Overview

**Target:** Closed beta / soft launch **June 8-12, 2026** (reset May 27 — slip of 3 calendar days from June 5-9 to account for scope additions: rate limiting separate Session 3.2.5 [2-3h], mobile experience Session 5.5 [3-5h], proxy auth rework Session 3.2 [4-6h]. Honest call now rather than 0.5-day slips later. Prior reset May 26 was June 4-8 → June 5-9 for OAuth scope; original May 20 reset moved June 1-5 → June 2-6.)

**Soft-launch framing (May 26):** "Closed beta" originally scoped as 5-7 invited friends. May 26 driver-chat decision: broader audience including non-close-friend testers (colleagues, peers, LinkedIn contacts), accounts persist forward into V1 as paid customers. **This changes the product posture meaningfully:** auth foundation built now persists, first impression matters more, account-method continuity prevents future migration friction. The "closed beta" label still applies but the scope is closer to a soft launch. Documented here so future-Marcelo (and Claude) doesn't make scope decisions against the original 5-7-friends framing that's been superseded.
**Path:** Y — backend infrastructure first, chat-driven UX rebuild defers
**Closed beta scope:** 5-7 invited testers, free, validate moat hypothesis
**Host architecture (locked May 21):**
- App: Netlify at `app.rolegarden.com` (production deploy in Day 7 Session 2)
- Proxy: Render at `rg-proxy-flpd.onrender.com` (live since Day 5)
- DNS: Cloudflare (live since Day 7 Session 1)
- Marketing site: Framer at `rolegarden.com` apex (post-beta, V1 work)
- Standard SaaS pattern (Stripe/Linear/Notion run app on subdomain, marketing on apex). Apex reserved for cold traffic, paid media landing pages, SEO, blog.
- Vercel was attempted May 20 and abandoned mid-flow (contradicted workdoc; see throwaway-deploy session summary).

---

## Sprint State — Day 6 Sessions 1-4 Complete

### What's Shipped (Days 1-5)

**Day 1 (May 4) — Layout + result cards**
- Unified top nav: 5 numbered stages (New Search · My Opps · Recruiter · HM · Presentation)
- Result cards visual redesign: score on left, /100 bars, source pills, no clutter
- Left rail rebuilt: titles chip multi-select, location autocomplete, recent searches accordion
- /10 → /100 for user-facing exports
- Fixed `returnclaudeHeaders` typo (was silently breaking scoring)

**Day 1.5 (May 8) — Source audit + debug**
- JSearch wiring confirmed (was always JSearch, just mislabeling as "LinkedIn")
- Dropped niche remote APIs (Remotive, RemoteOK, WWR, Wellfound, Jobicy, Arbeitnow)
- Renamed hardcoded `source: 'LinkedIn'` to use real `job_publisher` field
- Marketing line locked: "AI-curated jobs from LinkedIn, Indeed, and ZipRecruiter"

**Day 2 (May 9) — Onboarding + identity**
- 3-step onboarding (About you → Resume → Ready)
- User helper functions as single source of truth (`getUserFirstName/LastName/FullName/Email/etc`)
- 28 of 41 hardcoded MARCELO references replaced
- Demo opp renamed "Acme Software" → "Demo Opp"
- Path A discoverability: tip card + modal explainer with 3 numbered steps
- Sign out link in avatar dropdown

**Day 3 (May 10-11) — Render proxy + Yelp deliverable night**
- Render proxy deployed: `https://rg-proxy-flpd.onrender.com`
- Anthropic key moved to proxy env vars
- 14 Claude callsites wired through proxy via `claudeHeaders()` helper
- TA + HM prompts rewritten to match locked frameworks
- Streaming added to `wsRunPrep` main prep call
- Pitch prompt deferred (M1D Strategy Coach is post-beta)
- Yelp Senior Partner AE outreach exercise built and submitted

**Day 4 (May 12) — M1C HM rebuild**
- M1C HM two-phase flow: story selection → prep generation with embedded stories
- "No hallucinated STARs" principle codified
- STAR library: bulk import, auto-tagging, library page split from profile
- Path Y reset: backend infrastructure first, chat rebuild defers
- "AI-first chat-driven UX" principle locked

**Day 5 (May 12-13 night) — Supabase + JSearch proxy**
- Supabase project provisioned (Free tier, East US Ohio)
- 6-table schema created with RLS: users, resumes, star_stories, opportunities, searches, outreach
- Auto-create trigger on auth.users insert → users profile row
- Supabase auth wired end-to-end (sign up, sign in, sign out)
- New-user onboarding routing fixed
- JSearch proxy endpoint `/api/search` live
- "Connect Search Engine" UI hidden when proxy is active
- Search auth header bug fix
- Workdoc dedup: 149 cards → 38 unique (75% size reduction)
- Search Engine PRD written

---

**Day 6 Sessions 1-6 (May 16-17) — rgData write-through + Save for Later + Suppression filter + Onboarding intent capture + Intent in rail + source_job_id + saved-job re-scoring reuse + M1A-depth lock**
- **Session 1:** `rgData` module shipped — write-through cache wrapping Supabase + localStorage. Exposes `syncIntent`, `getIntent`, `saveForLater`, `removeSavedForLater`, `getSaved`, `getSuppressedJobIds`. Schema additions to `users` table (`industries text[]`, `target_titles text[]`, `target_companies text[]`, `status text`) applied
- **Session 2:** Result card 3-button footer (Create Opp / Save for Later / Skip). Saved-for-Later dedicated view with rail button + count badge. `rgData.saveForLater` enriches the local cache with score / strengths / gaps for re-render fidelity. Greyed-card treatment shipped dormant pending Session 3 decision
- **Session 3:** Pre-scoring suppression filter wired into `runM0Search` against opps + skipped jobs (closed-beta blocker for historical search dedup closed). `isSaved` render-time filter reversed — saved jobs now appear greyed in new searches per the search_engine_prd three-state model. Fixed `getSuppressedJobIds()` opp-side bug discovered mid-session
- **Session 4:** Onboarding widened from 3 to 4 steps. New Step 3 — Intent Capture form. Form-based interim — Constitution Principle 1 violation acknowledged in three places in code. `onboarding_prd.md` written
- **Session 5:** Intent visible in search rail (collapsible block above Recent Searches). Soft nudge banner for skipped-onboarding users. Rail-TITLES pre-populate from intent on first Discover render per session
- **Session 6 (closeout):** `opportunities.source_job_id` schema + wiring (forward-compat for Day 7+ opp sync). Saved-job re-scoring reuse in `runM0Search` (30-cap preserved, reused saves skip Haiku call). M1A-depth decision LOCKED B (hybrid expand-on-click, four guardrails)

**Deferred from original Day 6 plan:**
- Full Supabase read-replacement — write-through pattern in place via `rgData`; legacy localStorage reads still drive the app. Day 7+ infrastructure work
- Signal persistence (skip reasons → Supabase, opportunity stages → Supabase) → Day 8, pairs with cron's signal-reading code
- Search scoring prompt update to read intent fields → folded into Day 9-10 scoring rewrite

---

**Day 7 Session 1 (May 18-20) — Resend infrastructure + Cloudflare DNS migration**
- Resend account created, domain `rolegarden.com` verified
- Cloudflare DNS migration (from Namecheap) — unlocks Day 9 Path A inbound parsing and Day 11-12 production deploy CDN/SSL setup; pulled forward as side-benefit of resolving Namecheap MX constraint
- Cloudflare Email Routing live: `feedback@`, `noreply@`, `hello@` + catch-all all forwarding to `martinsmazza@gmail.com`
- `/api/send-email` endpoint deployed on Render `rg-proxy` (raw https + Resend API, no SDK, reuses `PROXY_AUTH_KEY`)
- End-to-end test send verified from `hello@rolegarden.com` to Gmail
- Auth-vs-product email split decision locked: `noreply@` for Supabase auth emails (no Reply-To), `hello@` with Reply-To `feedback@` for product emails
- Session ran 6h vs 30min scoped (Resend doc claim was wrong about MX-skip; forced replanning that pulled DNS infrastructure forward)
- Open: Supabase SMTP config + signup confirmation template + Site URL fix all roll to Day 7 Session 2

**Throwaway-deploy / friend-test session (May 20) — feedback captured, deploy not shipped**
- Vercel deploy abandoned mid-flow after contradiction with workdoc surfaced (workdoc specifies Netlify; brief specified Vercel — driver chat error)
- Friend testing happened in-person on Marcelo's machine instead — ~30min, pre-briefed about Marcelo-scoring contamination
- MARCELO audit surfaced three real exposures beyond the "13 cosmetic references" in Constitution checklist:
  1. `scoreJobs()` prompt hardcodes Marcelo's profile — every search by every user scored as if they're Marcelo
  2. `runM1` + Auto-Assess prompts inject Marcelo's resume summary verbatim
  3. `RESUME_TEXTS` constant ships Marcelo's four real resumes in code; `hasResume` always returns true; new users see "Tailor My Resume" CTA pointing to Marcelo's resume
- Friend feedback captured (multi-profile, two-resume engine, button rename, framework Q additions) — triaged into roadmap below
- Accordion regression discovered: TA + HM prep sections render but no longer collapsible — later (May 21) correctly diagnosed as never-built feature, deferred to V1 polish

**Day 7.5 — Closed-beta blocker cleanup (May 20-21) — FULLY SHIPPED**

Five sessions across two days. MARCELO contamination class verifiably eliminated across all live-reachable prompt sites in the closed-beta UX. Audit-scope-too-narrow pattern diagnosed and corrected with structural discipline (pre-swap + post-swap file-wide grep verification).

- **Session A (May 20)** — MARCELO prompt cleanup at 3 named sites: `scoreJobs`, `runM1`, `runAutoM1`. New helper `getUserContextForPrompt()` shipped as canonical context-injection pattern. `RESUME_TEXTS` constants emptied. `hasResume` guard fixed to read only from `localStorage.rg_resume_primary`. 4th site (`runM2`) discovered dead-code, deferred to Day 8.
- **Session C (May 21)** — Read-only audit of TA + HM prep prompts vs M1B + M1C locked frameworks. Surfaced 4 TA gaps + 1 HM gap + 4 cross-cutting findings. Most critical finding: TA + HM prep at `wsRunPrep` still using legacy `getBackground()` helper (Session A escape).
- **Session D (May 21)** — Six surgical fixes to TA + HM prep prompts: (1) MARCELO escape swap at TA + HM call sites, (2) added TA Section 1 "The Interviewer", (3) reshaped TA Section 2 to "Your Prep Before the Call" (friend-feedback May 20 fix), (4) TA Section 4 forward-compat language for Day 9-10 web search wiring, (5) reordered TA per M1B PRD + dropped Section 7, (6) anchored standard questions in both prompts. 7th MARCELO site discovered at M1D pitch, flagged.
- **Session E (May 21)** — M1D pitch MARCELO escape swap. Surfaced **audit-completeness gap**: prior session enumerations had missed 3 additional live-reachable contamination sites (`wsRegeneratePitch`, `addNotesAndPrep`, `buildPrepPrompt`). Build chat correctly retracted brief's "fully eliminated" overclaim.
- **Session F (May 21)** — Three final swaps at the audit-surfaced sites. Pre-swap + post-swap file-wide grep verification structurally embedded in brief. Closed the contamination class with grep evidence: zero live-reachable consumers of `getBackground()` remain. 5 dead-code sites remain on Day 8 cleanup list.

**Driver-chat PRD updates shipped May 21:**
- `m1b_recruiter_interview_prd.md` — Section 7 conditional STAR removed. M1B = 6 sections. Anti-pattern added: "Generating STAR stories in M1B output."
- `m1c_hm_interview_prd.md` — added "What does the next round look like?" to standard inclusions; added Deferred-to-V1 section (skills section, multi-profile awareness, 6-section accordions)
- `resume_engine_prd.md` — Two Resume Audiences section added (ATS vs Human); ATS-audience locked for closed beta; multi-profile architecture promoted to V1 priority

**Driver-chat decisions locked May 20-21:**
- **Result Card Depth: Option C** (rich default M1A-depth) with streaming render baseline — supersedes May 17 Option B hybrid lock
- **Marketing framing: truthful-future** ("RG learns from every job you save, skip, or apply to") with explicit gap window for closed beta
- **Web search infrastructure** required for M1B/M1C/M1D prep activation — Day 9-10 scope expansion, ~+2-3h, ~1 day launch slip
- **Skills section** — V1 cross-cutting addition, post-beta
- **Multi-profile architecture** — V1 priority, no closed-beta workaround
- **Button rename pass** ("Create Opportunity", "My Opportunities") — punt to post-beta CSS sweep

---

## Sprint Plan — Days 7-14 (Reset May 20-21)

### Day 7 — Resend integration + tailored resume PDF (multi-session)

**Sprint state (May 26):** Sessions 1 / 2 / 2.5 / 2.6 / 2.7 / 2.8 / 2.9 diagnostic / 2.10 / 3 / 3.1 SHIPPED (May 18-26). Sessions 3 + 3.1 = Search rail Section 1 + render fix. Session 3.2 = architectural pivot (remove Intent step, remove STATUS, welcome copy, tooltip). Session 4 = Saved Searches with save+auto-search-config + 2-cap. Session 5 = Settings restructure (simplified, no Default Search section). Session 6 = M accordions + Forgot Password fix. Sessions 7-9 = resume engine. Day 9 = OAuth Google + Microsoft. Day 10 = scoring rewrite + web search + Option C + direct-employer URLs.

**Session 3.1 (SHIPPED May 26) — Render Fix + Hydration:** Surgical fix to Session 3 escapes. P0: added missing `rgRailIntentRenderChips(listName, containerId)` function called from `rgRailIntentRender()` lines 11960-11961 but never defined (Session 3 namespace rewrite deleted it but kept the calls). P1: onboarding LOCATION field autocomplete pattern, Supabase 400 diagnostic on users upsert, claudeHaiku x-proxy-auth header. Browser-load test verified zero console errors before declaring shipped. **New discipline class added to Constitution:** every session touching >50 lines of JS requires browser-load verification — open deployed file, navigate affected surfaces, watch console for zero errors. Node.js `--check` necessary but not sufficient.

**Architectural pivot (May 26):** Onboarding Intent step REMOVED. User-level intent fields (industries, target_titles, target_companies, location) are no longer captured in onboarding. Instead, saved searches become canonical intent surface — each saved search has its own criteria, naming, optional auto-search toggle. Rationale: (1) onboarding-captured intent gets stale within weeks (users don't update Settings preferences they set once at signup); (2) multi-intent natively supported — user looking for Sales AE + Marketing Manager + different geographies = two saved searches with different criteria; (3) collapses V1 multi-profile architecture into closed beta naturally; (4) tests the actual product loop (does saving + naming + receiving auto-search emails create value?); (5) closed-beta sprint discipline still holds — pivot adds ~1-2h net work, no launch slip if executed cleanly. **STATUS field also removed** — Active/Open/Exploring had no clear use case in new architecture (auto-search cadence becomes per-search frequency setting instead). Logged to V1 marketing reconsideration: status may return for marketing segmentation if needed.

**Session 3.1 (SHIPPED May 26) — Render Fix + Hydration:** Surgical fix to Session 3 escapes. P0: added missing `rgRailIntentRenderChips(listName, containerId)` function called from `rgRailIntentRender()` lines 11960-11961 but never defined (Session 3 namespace rewrite deleted it but kept the calls). P1a: onboarding LOCATION field autocomplete matching rail pattern (5 new helper functions, ~80-90 lines). P1b + P1c surfaced as diagnostics (out of scope to fix this session): Supabase 400 (likely unrun schema migration; Marcelo verified ran "Success. No rows returned") + claudeHaiku 401 (pre-existing per-browser localStorage credential pattern, real product debt). Build chat operationalized the undefined-reference audit discipline as Python tooling: enumerates all function defs + all calls + subtracts JS builtins denylist + manual triage of remainder. 473→474 defs after P0 fix, 102 unknowns triaged. **The new audit discipline is now operationalized tooling, not just a discipline acknowledgment.** Two significant out-of-scope findings: (1) pre-existing Constitution Rule #3 violation at lines 7142-7152 — `_origSetEngageSub` override pattern, predates Session 3, Day 8 cleanup; (2) 45 dead-code candidates (definition-only matches, mostly false positives but `fetchRemoteOK`/`fetchWWR`/`replaceM1Field`/`saveToNotepad` worth Day 8 investigation). File: 13,317 → 13,431 lines (+114 net).

**Critical finding from Session 3.1 (May 26) — `mt_proxy_auth` is a real closed-beta blocker:** P1c diagnostic concluded: *"the proxy-auth-via-per-browser-localStorage UX doesn't scale beyond the founder. Every new user must manually paste a key before any Claude calls succeed."* Given May 26 scope re-characterization of closed beta as soft launch to broader audience with account continuity to V1, this is a real new scope item. **Three options:** (a) fix proxy auth architecture before closed beta — Supabase-session-bound proxy auth, ~3-5h engineering; (b) manually paste auth for 5-7 invited testers — works for tight invite list, doesn't scale to broader soft-launch audience; (c) onboarding email with browser-console snippet — hacky, breaks credible first impression. **Decision pending — likely option (a) given soft-launch broader audience scope.**

**Session 3.2 (SHIPPED May 27) — Proxy Auth Architecture Rework:** Replaced per-browser `mt_proxy_auth` localStorage credential with Supabase-session-bound auth across client (`index.html` 13,431 → 13,440, +9 net) and server (`rg-proxy/proxy.js` 323 → 377, +54 net; `package.json` 12 → 15, +3 net adds `@supabase/supabase-js`). All 4 curl tests passed (valid JWT → Claude response; missing auth → 401; invalid JWT → 401; health check → dual-auth model confirmed). Browser-load smoke test passed: sign-in → onboarding → Search render with zero console errors. **Hard cutover successful** — no backwards-compat fallback in proxy. The 14-callsite async cascade was a non-issue because every enclosing function was already async — prior discipline paid the cost. `/api/send-email` deliberately kept on shared-secret auth (server-to-server use case from cron + Path A V1) — `PROXY_AUTH_KEY` env var preserved on Render. **Build-chat discipline matured:** pre-flight grep enumeration A-D ran cleanly, deploy sequence pre-written with curl commands, failure modes anticipated with diagnostics, rollback path documented. **Critical discovery during smoke test:** M1A "Ask Claude" chat lacks current-render-state context — when user asks Claude to rewrite an outreach message, Claude asks user to re-input information already on screen + exposes internal field IDs (`out-ref-ask`, `out-cold`, etc.). Closed-beta scope decision: added to Session 6 (M Accordions session). **Deploy hiccup logged:** proxy.js committed before package.json caused 1 failed deploy (~5 min recovery); discipline lesson added — multi-file proxy deploys need dependencies-first commit order OR single-commit local git push.

**Session 3.3 (SHIPPED May 27) — Architectural Pivot Implementation:** Onboarding Intent step removed (Resume → Ready, 2-step flow). STATUS field removed from rail + Supabase schema (DROP COLUMN status). Welcome message expanded (🌱 64px, "Welcome to Role Garden." 32px, new body copy). TARGET COMPANIES tooltip added (new `.rg-tip` / `.rg-tip-bubble` pattern with hover + tap support + document-level click dismissal). File: 13,440 → 13,142 lines (-298 net, deletion-dominated). 21 deleted function definitions enumerated cleanly. 25 distinct touch points for Intent removal — exceeded brief's stop condition (~10 sites) but build chat surfaced and proceeded with full enumeration documented (single namespace, no shared callers outside Intent surface = safe). DOM ID stability preserved (`ob-step-2` / `ob-step-4` kept, visible "1"/"2" diverge from DOM IDs — Session 2.6 precedent). Browser-load smoke test passed: new user signup → 2-step onboarding → empty Search rail with no STATUS, no console errors. Build chat surfaced 8 V1/Day 8 cleanup candidates including orphaned mobile-share modal + functions (kept intact this session, Day 8 sweep), `.ob-status-btn` and `.rg-mobile-share-tip` orphaned CSS classes, `OB_INDUSTRY_SUGGESTIONS` mis-named constant (rail still uses it), `obGoStep3` misleading name, tooltip keyboard-accessibility gap, and the 🌱 emoji vs Constitution rule tension (driver chat decision: A — Constitution exception for brand mascot leaf, V1 polish for clean separation).

**Session 3.4 (SHIPPED May 27) — Ready Step Removal + Copy Fixes + Header Logo Resize:** Three surgical changes completing the architectural pivot. (1) Ready step removed entirely — resume upload (or skip) auto-routes through `obGoStep3()` → `obFinish()` → land directly on Search rail. Single funnel for happy path + skip path. `obFinish` toast "Welcome [name] — let's find your next role" IS the new "you're all set" moment. (2) Copy fix: "Your career, cultivated. Let's get you set up in 3 steps." → "Your career, cultivated. Upload your resume to get started." (build chat chose hybrid that preserved brand tagline while fixing false-claim). (3) Header logo 26×26 container + 14×14 SVG → 32×32 container + 18×18 SVG (matches existing onboarding card header pattern for proportional consistency). Step indicator HTML deleted entirely (single-step doesn't need numbered progress). `obSetStepActive` and `_obAdvanceToReadyStep` functions deleted (no remaining callers). File: 13,142 → 13,105 lines (-37 net). Build chat reflection: 40 lines of session-marker comments added back while deleting ~78 lines of code; flagged 2 markers as arguably redundant ("discipline is ONE central history comment per deleted region"). Smoke test: new user signup → single-step onboarding → land directly on Search ✅, console clean ✅. **Regression check surfaced finding:** existing user `+rg3@gmail.com` lands on Profile/My Resumes instead of Search after sign-in. Likely pre-existing routing debt in `authOnLoginSuccess` for already-onboarded users — bundled into Session 3.5 scope (~30 min add-on). Build chat's honest reflection noted `obShowStep` is now vestigial single-step + orphaned mobile-share modal is fully orphaned post-3.4 — both Day 8 cleanup candidates.

**Session 3.5-R (SHIPPED May 28) — Resume PDF Extraction Fix + Variant-Label Removal (CLOSED-BETA BLOCKER, CLEARED):** Replaced homegrown PDF byte-scraper with PDF.js 3.11.174 shared `extractPdfText()` helper + `loadPdfJs()` lazy-load + `class UnreadablePdfError` + `disableWorker:true` retry fallback. Found 3 byte-scraper sites (not 2) — resume upload (`obProcessFile`), STAR import, AND `processResumeFile` (My Resumes re-upload) — all consolidated into the existing `extractTextFromFile` helper via PDF.js. DOCX latent byte-scrape bug fixed for free (mammoth.js). Honest-failure path: never saves garbage, shows brand message, Next stays disabled. Removed M1A variant-label injection (both paths) + `resume_used` JSON + UI badge chips + email filename/subject/body variant refs. `RESUMES`/`autoSelectResume` LEFT DEFINED (Day 8 deletion). M1C label injection deliberately LEFT for 3.5-M (entangled with `resume.summary`). File 13,105 → 13,159 (+54), then +`pk` one-line fix. **`pk` fix (Option A):** line ~4513 `updateKeyStatus` referenced deleted `pk` var (Session 3.2 leftover) → ReferenceError on API-keys-modal `oninput`; fixed to proxy-mode-always-true string. **SMOKE TEST RESULTS:** (1) martinsmazza re-upload PDF → `rg_resume_primary` clean text, 92/100 real assessment (was 45/100 corrupted), no Player-Coach chip ✅; (2) fresh incognito user + DOCX → clean extraction, onboarding appeared (no bleed), 88/100 real assessment ✅; (3) honest-failure proven headless ✅. PDF + DOCX paths both confirmed live. Blocker CLEARED. **Surfaced for follow-up:** (a) `c.resume` now `undefined` at legacy `runM2`/`runM3` reads (~L3504/3580) — left deliberately, these are 3.5-M's body; (b) `sendAutoAssessEmail` subject still says "MatchTalent" (pre-beta brand fix); (c) localStorage-bleed/data-isolation gap (logged in Session 3.5); (d) Day 8 dead-code: `RESUMES`, `autoSelectResume`.

**Session 3.5-M (SHIPPED May 28) — Complete getUserContextForPrompt Migration (Principle 2/3 blocker CLEARED on live surfaces):** Pre-flight grep CORRECTED the brief: 3 of the 4 named targets were DEAD code (sendM1Chat, runM2, runM3 — bootstrap-unreachable loops), and the brief MISSED the 3 live generators actually carrying the bug — `runTAPrep` (M1B, L10704), `runHMPrep` (M1C, L10746), `runPitchPrep` (M1D, L10779) — plus the live Ask-anything chat (`askCardClaude`, L8348). Marcelo chose Option 1 (live-only). Migrated those 4 live sites: `getBackground()`/`getUserBackground()` → `getUserContextForPrompt()`, preserving STAR-append where it existed. Caught + fixed a latent double-STAR bug in M1C. Raised slice budgets (600→1100-1200 prep, 500 chat) because the context-block header was eating the old budget and clipping real resume content (intentional, not a blind swap). Browser-tested headless with a NON-Marcelo resume (Priya, Pediatric ICU RN): all 4 surfaces cite real resume markers, ZERO Taboola/120%/Supermetrics persona leakage; empty-user honest-fallback also clean. **Left defined (Day 8 deletion):** all dead code (sendM1Chat/m1Chat* family, runM2/runM3/renderM2/renderM3/renderModel-dispatch/swMod/M2S/M3S, RESUMES/autoSelectResume/c.resume reads) + `GENERIC_BACKGROUND_FALLBACK`/`getBackground`/`getUserBackground` (still have dead callers; retire when dead path deleted Day 8). Hardcoded Supermetrics pitch + Marcelo STARs live only in dead runM3/runM2 — die with them Day 8. File 13,161 → 13,179 (+18).

**Session 3.5 (SHIPPED May 28) — Main Nav Restructure + Font Sizing + Routing Fix + localStorage Reconciliation:** All 4 items shipped. (1) Nav: removed `.rg-num` badges + SVG icons from New Search & My Opportunities (now clean text-only), dropped numbers from prep stages too, added inline "Interview Prep" group label (NOT a dropdown), relabeled Recruiter Interview→Recruiter / Hiring Manager Interview→Hiring Manager / Presentation Interview→Presentation. All onclick wiring preserved (navGo/setEngageSub). (2) Font: `.rg-tab` 12.5/600→15/700; new `.rg-tab-primary` 16/800 on the two primary actions (matched weight); `.rg-prep-stage` 14/600 (a notch lighter). Still a nav bar, not pills. (3) "+" placeholder (`esn-add`, `onclick="return false;"`) grouped with prep stages — true no-op, no alert, doesn't throw; real frameworkless prep session = Session 3.6. (4a) Routing fix: removed `_onboardedLanding` conditional entirely (was `showProfile()` for onboarded) — ALL authed users now land on Search via `showDiscover()`; `checkOnboarding()` still layers onboarding for new users; boot-path (authBootGate) verified no cascade. (4b) localStorage-bleed reconciliation via `rg_active_uid` marker at top of `authOnLoginSuccess`: if authenticated UID ≠ stamped UID, clear prior user's scoped keys (resume, onboarded, profile, mt_v3, searches, STARs, name/email) + restamp; same UID = no-op; unmarked browser = trust+stamp. **Build chat made 2 correct judgment calls beyond brief:** (i) did NOT clear stamp on logout (brief said to) — clearing weakens the guarantee; persistent stamp means a different user always mismatches→always clears (strictly stronger); authSignOut confirmed not to wipe scoped keys, so login-side check is the single correct enforcement point. (ii) removed the routing conditional entirely vs just flipping the call. **Honest residual gap:** first cross-user login on a pre-marker browser isn't caught (can't know whose data is there → trust+stamp); every mismatch after is caught. Low closed-beta risk. File 13,179 → 13,233 (+54). Headless-tested: 16 nav assertions + 4 routing scenarios, 0 console errors. Day 8: orphan `.rg-num`/`.rg-prep-label` CSS cleanup.

**Session 3.6 (SHIPPED May 28) — Frameworkless "Prep" Tab (LEAN cousin path):** The "+" nav button (esn-add) became a real per-opp frameworkless prep tab — chat + output, no framework, no auto-kit, no carry-chain. Stayed lean: ~10 minimal-addition edit sites, did NOT thread `prep` through all 30+ ta/hm/pitch ternaries (most resolve correctly via `||` fallbacks / null-carry arms; added a `prep` case only where the fallback was wrong or would break). **Build chat caught 2 real correctness bugs the naive lean build would've shipped:** (1) prep work-state refine (isPitch=false) would've fallen into the TA/HM surgical-patch path and CORRUPTED `c.m.hm` — gave prep its own refine branch (full regenerate via wsRunPrep, not patch); (2) opening prep would've read `c.m.pitch` via ternary fallthrough — added prep's own `c.m.prep` lookup. Also added esn-add to rgSetActiveStage clear-list (brief missed it — highlight would set but never clear), edited `_origSetEngageSub` override IN PLACE (no new Rule-#3 layer). Frameworkless prompt: no imposed sections, model shapes prep to whatever stage the user describes in chat, grounds via `getUserContextForPrompt()` (3.5-M helper, NOT getBackground), honest fallback. Chat persists `rg_chat_${aid}_prep` (free from existing pattern); output saves `c.m.prep`. Verified: 29/29 headless, non-Marcelo grounding (Priya RN, zero persona leak), `c.m.hm` not corrupted on refine, 3-tab regression clean, empty-user honest fallback. File 13,233 → 13,306 (+73). **3.7 handoff:** prep currently has NO carry in/out by design — when 3.7 builds the shared notes pool, decide prep participation (build chat's rec: yes — frameworkless prep notes should feed forward). `_origSetEngageSub` Day 8 refactor must preserve `ta|hm|pitch|prep`→enterWorkspace routing.

**Session 3.7.1 (SHIPPED May 28) — Notes Pool Simplification (blob + auto-save):** CORRECTION to 3.7. 3.7 built an over-engineered "attributed pool" (array of `{text, source, ts}`, per-note edit/delete UI, "Add note" button, source-tag chips) — Marcelo confirmed with screenshot evidence this was over-built (root cause: driver-chat brief misread Marcelo's earlier instruction "without the button and icons but with the saved"). Actual model: ONE shared notes BLOB per opp, no attribution, auto-save on input, `✓ Saved` pulse each time. 3.7.1 simplifies to that. **Store shape:** `c.m.notes` = string (overloaded — same name, shape changes; built chat defended this vs new `c.m.notesText` field per Constitution "two ways to do it" debt warning). Marker `_notes_migrated:1 → :2`. **Migration handles 3 input states idempotently:** already-string (no-op), 3.7 array (sort by ts, join with `\n\n`), pre-3.7 legacy per-stage strings (fold + delete slot). Multi-state mix folds together, legacy first then ts-sorted array. **Removed all 3.7 attributed-pool artifacts:** `RG_NOTE_SOURCE_LABELS`, `rgAddOppNote`, `rgEditOppNote`, `rgDeleteOppNote`, `rgRerenderOppNotesCard`, `rgOppNoteAddFromInput`, `rgOppNoteToggleEdit`, `rgOppNoteDelete`, per-note DOM ids, .source field reads. **Added:** `rgSetOppNotes(c, text)` single writer, `rgOppNotesOnInput(textareaEl)` 300ms debounced auto-save, `_rgNotesSaveTimer` module-scope. **UI:** one textarea per tab bound to same blob, oninput auto-save, ✓ Saved pulse fires each save, "Shared across all prep tabs" as quiet subtitle (no counter), placeholder "Notes for this opportunity — what happened on calls, what you learned, what to remember. Visible in every prep tab." **Prompt feed:** `rgPoolPromptBlock(c)` returns raw blob with lead-in "[Candidate notes for this opportunity:]" (no per-entry attribution); empty/whitespace blob → empty string (skip prefix). **Build chat caught 2 correctness bugs the brief missed** — both whitespace-trim semantics issues from the array→string shape change: `hasPoolNotes` falsely reporting "has notes" for whitespace-only string (fixed to `!!rgGetOppNotes(c).trim()`), `rgDraftFollowup` empty-guard letting whitespace through to email prompt (fixed to `!notes||!notes.trim()`). **Browser-load verification RAN this session** (3.7 skipped it, Constitution gap closed): Playwright + headless Chromium, every brief verify item green (cross-tab visibility, edit propagation across all 4 tabs, migration of 3.7 array shape, prompt feed, follow-up email draft, no Add button / no Tagged label / no per-note rows, persistence end-to-end). 3 console errors all attributed to sandbox egress (Supabase 403), not 3.7.1 code. File 13,400 → 13,322 (-78 lines — real simplification, 192-line block replaced by 113 lines + 2 bug-fix one-liners). **Lesson:** driver-chat misread the original UI instruction and built a brief around the wrong model. Build chat executed correctly per brief; the error was upstream. Real rework cost: ~1 session. Surfaces a discipline: when user gives a brief design instruction with reference imagery, the driver-chat should restate the model in concrete terms BEFORE writing a brief, to catch interpretation errors before code lands.

**Session 3.7.2 (SHIPPED May 28-29) — Two follow-up fixes to 3.7.1 (rgDraftFollowup helper swap + persistent Saved indicator):** Live smoke surfaced two issues. (1) **Latent 3.7 bug — wrong model helper:** `rgDraftFollowup` called `claude(prompt, 800)` — the JSON-extracting helper (L4726) that regex-matches `{...}` in the response and throws "Could not parse response" when there isn't one. A follow-up email is plain prose, not JSON. Correct sibling is `claudeText` (L4749). Bug was wired before 3.7, stayed wrong through 3.7.1 because 3.7's headless verify exercised the data feed but only checked what went into the function, not what it did with the response. **Fix:** one-line swap `claude` → `claudeText` + inline comment to prevent re-introduction. `rgDraftFollowup` is the only `rgDraft*` function (grep-confirmed), no other prose paths affected. (2) **Persistent Saved indicator (UX rework):** 3.7.1's `✓ Saved` was a transient ~1.4s pulse that faded out, leaving the textarea looking unsaved between pulses. Reworked to a persistent two-state indicator on `#opp-note-saved-flash`: `.is-saving` (grey, `--ink3`) flips on instantly when user types; `.is-saved` (green, `--brand`) flips on when 300ms debounce fires and `rgSetOppNotes` returns. Both states stay rendered, no fade/teardown. On render with non-empty blob: `.is-saved` from the start (returning users see solid green). **Empty-blob judgment call:** no indicator at all — grey would imply in-flight save with nothing to save, green would imply content state when there isn't any; hidden is cleanest, indicator appears the moment user types. SVG inherits via `stroke="currentColor"` so checkmark glyph flips with text color. File 13,321 → 13,345 (+24). Syntax clean. Live verify pending Marcelo's reload. **Lesson logged:** verification that doesn't traverse the actual response-handling path will miss this class of bug — headless tests of "what goes in" without "what comes out" are insufficient for prose-vs-JSON helper mismatches.


- Save-search flow with naming + auto-search toggle + frequency setting
- Display saved (named) searches first, recent (auto-snapshots) second
- **Auto-search cap: 2 enabled max in closed beta/free/trial; unlimited in paid V1**
- Saved searches UNLIMITED. Pricing leverage maps to active automation, not storage.
- Question-mark tooltip on Save Search action
- Post-cleared-cards empty state copy
- **NEW: Left rail restructured as NAV** — Search / Saved jobs / Recent searches as visually distinct accordion sections (not buried sub-elements below Search button). Larger font hierarchy. Each section collapsible. Better information architecture for closed-beta intuitive use.
- ~4-5h (was 3-4h, expanded for rail nav restructure).

**Session 4.5 — INDUSTRIES Autocomplete Pattern (NEW May 27):** INDUSTRIES input currently uses chip-grid "common picks" mixing industries with job functions (Hospitality alongside Sales — confusing). Modern UX (per Marcelo's LinkedIn + Indeed reference screenshots) uses typeahead suggestions matching user input. Replace common-picks chip grid with typeahead pattern matching the rail LOCATION autocomplete pattern shipped in Session 3.1. ~2h.

**Session 5 — Settings Restructure (simplified):** Account + Auto-Search defaults sections only (no Default Search section per architectural pivot). ~1.5h.

**Session 5A — Rate Limiting 3-Tier + Email Alerts** (renamed from "3.2.5" — execution position 5A clearer): Per Marcelo May 26 evening pushback on hard hourly caps. 3-tier approach:
- **Tier 1 — Per-IP rate limit (strict, hidden):** ~100 requests/hour per IP for any API call. Catches bots/scrapers. Returns 429 silently.
- **Tier 2 — Per-user soft cap with email alerts:** ~100 Claude calls per user per day. At 80%: silent log. At 100%: silent log + email to Marcelo. **No 429 to user.**
- **Tier 3 — Per-user hard cap (only clear abuse):** ~500 Claude calls per user per day. Friendly message.
- Email alerts via existing Resend infra.
- ~2-3h estimated.

**Session 5B — Mobile Experience** (renamed from "5.5"): Closed-beta blocker. Per Marcelo May 27 lock (Q-Mobile A): mobile shows Search + My Opportunities (= M1A workspace). M1B (recruiter prep), M1C (HM prep), M1D (presentation prep) blocked on mobile with "Available on desktop" message + "Continue in desktop →" CTA. M1A allowed because outreach copy is genuinely mobile-actionable. Search rail collapsed into accordion pattern. Test on iPhone Safari + Chrome at minimum. ~3-5h.

**Session 6 — M Accordions + Forgot Password UX Fix + M1A Chat Context Fix** (renamed and expanded May 27):
- **Per-section collapsible accordions for M1A/M1B/M1C/M1D output** — build chat asks Marcelo about section names during execution. Empty accordions render with section names BEFORE generation runs ("framework as scaffolding"). ~3-4h.
- **Forgot Password UX fix** (renamed from "mobile fix" — issue is universal not mobile-specific). Empty/invalid email submit fires green "reset email sent" alert as false positive. Fix: (a) client-side validation BEFORE `resetPasswordForEmail()` call rejects empty/malformed email with inline message, (b) success messaging to security-correct framing ("If an account exists for this email, you'll receive a reset link shortly") — avoids leaking user existence per standard auth pattern, (c) proper error handling for Supabase rejection (network errors etc.) with red alert. ~30-45 min.
- **Native `confirm()`/`alert()` dialog replacement (NEW May 28 — surfaced during 3.5-R account cleanup).** App uses raw browser `confirm()`/`alert()` dialogs in places (e.g. "app.rolegarden.com says — Remove your uploaded resume?" on the My Resumes Remove action). These show the unprofessional "app.rolegarden.com says" chrome and break brand. Replace ALL native `confirm()`/`alert()` calls with the app's own branded modal/toast pattern. Grep `confirm(` + `alert(` to enumerate; likely several sites (resume remove, opp delete, reset-onboarding, etc.). ~1-1.5h. (Could alternatively fold into Session 3.5 nav/polish — driver-chat call when 3.5 brief is written.)
- **`otp_expired` / expired-email-link handling (NEW May 28 — surfaced during 3.5-R account cleanup).** Expired Supabase email links (confirmation / magic-link / password-reset) dump the user back to the app with a raw URL hash `#error=access_denied&error_code=otp_expired&error_description=Email+link+expired` and NO friendly in-app handling. Add: detect the `error_code=otp_expired` (and related auth errors) in the URL hash on load → show a branded "This link expired — request a new one" message with a re-send CTA → clean the hash from the URL. Same auth-UX-cleanup family as the Forgot Password fix. ~30-45 min.
- **M1A Chat Context Fix** (NEW May 27 — discovered during Session 3.2 smoke test). Issue: when user opens "Ask Claude" inside M1A workspace and asks to rewrite an outreach message, Claude responds asking user to re-input information that's already in M1A render (the 4 outreach messages, the opp details, the resume context). Exposes internal field IDs (`out-ref-ask`, `out-cold`, `out-ref`, `out-ta`) instead of friendly names. Root cause: chat system prompt doesn't include M1A render state. Fix: wire current M1A render state (opp details + 4 outreach messages + resume summary + friendly naming) into the chat's system prompt when "Ask Claude" opens. ~1-2h.
- **Total estimated:** ~8-9h (was ~6-7h; +native-dialog replacement + otp_expired handling, both surfaced May 28 during 3.5-R account cleanup; the dialog-replacement could move to Session 3.5 nav/polish instead).

**V1.5 mobile evolution (logged):** Search rail retracts to left drawer; card takes full mobile screen; swipe left/right between cards (Tinder-style mobile-native UX). M-tabs may extend to mobile per V1 evaluation. ~6-10h V1.5 work.

**Session 2.7 (SHIPPED May 22) — Name-Field Consumer Audit + Fixes:** Pre-flight grep surfaced 27 name consumers (vs brief's 5-15 estimate). 3 broken sites fixed: (a) Settings → Account "Full name" → side-by-side First / Last fields with canonical-key read/write + dual write to `p.firstName`/`p.lastName` for round-tripping, (b) Settings → Account Email field made read-only with population from Supabase auth + "Email changes via support during closed beta" helper text, (c) profile-icon initials — root cause was `authOnLoginSuccess()` never called `updateUserUI()` (render function was already correct). Fix #3 expanded by ~10 lines (per brief's <30-line stop condition) to also wire metadata → localStorage sync in `authOnLoginSuccess` — closes a latent bug for returning users on new devices or after browser-storage clear. M1A/M1B/M1C/M1D prep prompts audited for MARCELO contamination — verified clean (10 `Marcelo` references in file are all comments documenting prior Day 7.5 cleanup work, one runtime reference at line 12628 is intentional closed-beta invite copy). File: 13,212 → 13,287 lines (+75).

**Session 2.8 (SHIPPED May 22) — Signup/Onboarding Routing Regression Fixes:** Three bugs surfaced from fresh-signup smoke test against production. Build chat's pre-flight grep refuted 2 of 3 brief hypotheses; declined to ship speculative fix for Bug 1 because code reads correct; diagnosed routing race in `authOnLoginSuccess` as the actual bug behind "landed on My Resumes" symptom. Fixes: (1) defensive `rg_onboarded` clear at `authOnLoginSuccess` start — fresh-signup never inherits stale `'1'` flag from prior smoke-test sessions in same incognito window (~10 lines), (2) routing race fix — pre-fix called `showProfile()` unconditionally BEFORE the onboarding gate check; post-fix runs onboarding gate first, then routes to Discover OR onboarding overlay (~14 lines), (3) Profile-page LinkedIn input + `saveProfileFields()` removed entirely — closes FLAGGED LEGACY surface from Session 2.7 audit AND closes the browser-autofill exposure that was rendering email value in the LinkedIn URL field (-7 lines net). Bug 1 (names not in localStorage) **declined as no-code** — code at `authSubmit` lines 4046-4096 reads correct; most plausibly test-environment contamination from cross-context incognito tab reuse. Retest in fresh incognito window deciding evidence. If retest reproduces, Session 2.9 adds Supabase users-table profile-row fallback to `authOnLoginSuccess` metadata sync (~15 lines). File: 13,287 → 13,304 lines (+17 net).

**Session 2.9 (SHIPPED May 22) — DIAGNOSTIC-ONLY: rg_onboarded Premature Write Investigation:** Read-only session. Pre-flight grep expanded to bare-token + variable-keyed setItem patterns + alternate identifier names — **confirmed Session 2.8's finding intact**: no code path in index.html sets `rg_onboarded='1'` on fresh signup. Brief's "audit-scope-too-narrow" hypothesis refuted. Boot-time execution path traced. **Incidental P0 security finding surfaced:** lines 8855-8857 of index.html contained Marcelo's REAL production Apify API token + REAL LinkedIn `li_at`/`JSESSIONID`/`bscookie` auth cookies + user-agent string, hardcoded into bundled JS and seeded to every visitor's localStorage on page load. Live exposure on `app.rolegarden.com`. Diagnostic also identified the cross-context email confirmation flow as the most-likely root cause of the `rg_onboarded='1'` symptom (user signs up in incognito browser A → email link opens in default browser B with pre-existing localStorage from prior dev work, including stale `rg_onboarded='1'` flag). No code changes this session. File unchanged at 13,304 lines.

**Session 2.10 (SHIPPED May 22) — P0 Security Cleanup + P1 Cross-Context Routing Fix:** Two fixes, P0 first. Apify path scope much broader than initial brief estimated — 17 distinct call sites + 32 grep matches enumerated. **P0:** deleted entire Apify + LinkedIn-cookie code path. Helpers (`getApifyKey`/`getLiCookies`/`getLiUserAgent` + duplicate `getApifyKey` stub in V8 ADDITIONS block), function-body portions of `loadKeys`/`updateKeyStatus`/`checkKeys`/`saveKeys` (preserved Anthropic/Proxy/Google/RapidAPI surfaces), `runApifyActor` + `fetchRemotive` (zero callers but contained only live `runApifyActor` reference), boot-init triplet at lines 8854-8857 (THE LEAK), `runSearch` apifyKey marker, ERROR_FIXES['apify'] entry, hidden compat HTML (`apifyKey` input + `apifyStat` div), help-modal "Apify keys" copy. Defensive `localStorage.removeItem('mt_apify'/'mt_li_cookies'/'mt_li_ua')` scrub block added in place of boot init — proactively cleans returning visitors' cached creds. **Pre-rotation credentials revoked by Marcelo before session start:** Apify token deleted from Apify dashboard, LinkedIn signed out everywhere. **P1:** added `created_at`-based fresh-user defensive clear in `authOnLoginSuccess` (Option A from Session 2.9 diagnostic) — gate checks user is within ~5 min of created_at AND has no resume/opps/searches; if true, clears `rg_onboarded`. Returning users unaffected; cross-context email confirmation now correctly forces fresh users through onboarding. Collateral cleanup: pre-existing undefined-`lic2`-variable bug fixed via `liStat` block deletion. New audit-discipline class logged: **hardcoded literals audit** — future sessions adding infrastructure credentials must grep for credential patterns (`apify_api_`, `sk-ant-`, `eyJ...` for JWT) before shipping. Day 8 cleanup expanded with: help-modal staleness sweep (MatchTalent.ai refs, `matchtalent_v6.html` filename, emojis against brand), `fetchLinkedIn`→`fetchJSearch` rename, delete dead `fetchRemoteOK`+`fetchWWR`, dual-`getGoogleClientId` audit. Build chat acknowledged tool-budget inefficiency (audit-then-edit-granularly burned more calls than needed; restructured second turn for bulk edits). File: 13,304 → 13,282 lines (−22 net, deletion-dominated).

**Sessions 3-5 — Search Rail Rework (May 22):** Restructure the rail so search inputs + intent fields are a single canonical surface; rename Recent Searches → "My Searches" with naming + input-summary display; Settings restructure (Account / Auto-Search / Default Search). Foundation for V1 multi-profile (no `resume_id` in closed beta; additive at V1). Detailed scope in `features/search_rail_rework_brief.md`. NOTE: brief updated May 26 to use "Search" terminology consistently (was inconsistently "Discover").

**Session 3 (SHIPPED May 26) — Rail Section 1: Canonical Search Inputs:** Restructured top of rail to 5 canonical fields (TITLES, LOCATION, INDUSTRIES, TARGET_COMPANIES, STATUS) + Search button. Deleted "Your Search Profile" collapsible block entirely (~325 lines across CSS/HTML/JS). Fixed Day 6 Session 5 broken hydration — root cause was `rgRailIntentBridgeTitlesOnce()` setting per-session flag unconditionally at function entry BEFORE checking if intent had data (timing race during first sign-in latched flag with empty state). New `rgRailIntentBridgeOnce(intent)` passes hydrated intent explicitly and only locks the flag after evidence intent has loaded. 5 scenario smoke tests verify the fix. Added LOCATION field to onboarding Intent step (position 2: after status, before industries) + Supabase users.location column + getUserContextForPrompt scoring context. **Design call surfaced:** TITLES chip box (top of rail) treated as canonical surface for intent.target_titles — NOT two separate chip boxes — avoids reintroducing the two-place mental model. Driver chat confirmed May 26: ✅ correct call. **Constitution Rule #3 self-correction:** build chat first-pass used `const _origRender = ...; function = function(){_orig(); ...}` wrap pattern AND IIFE wrapper for `rgAddChip`/`rgRemoveChip` — caught both on its own review, refactored to patch the bottleneck (`rgRenderChips`) directly. Build chat now audits its own work against Constitution rules without prompting. File: 13,282 → 13,317 lines (+35 net).

- Session 4 (NEXT): My Searches naming + enriched display (~2-3h)
- Session 5: Settings restructure (~2h)
- **Estimated remaining:** ~4-5h combined

**Sessions 7-9 — Resume Engine (was Sessions 6-8, shifted May 26):** Resume parser → tailoring engine → PDF generation
- Session 7: Resume parser (PDF/DOCX/TXT → text → structured JSON schema)
- Session 8: Resume tailoring engine (summary tailor prompt + tailored HTML template — ATS/system version per friend feedback)
- Session 9: LightningPDF integration + PDF generation + Email 2 (outreach package + tailored resume attachment) wiring
- **Estimated:** ~5-6h combined

### Day 7.5 — Closed-beta blocker cleanup (May 20-21) — ✅ FULLY SHIPPED

**Goal achieved:** MARCELO contamination class fully eliminated across all 9 live-reachable prompt sites in closed-beta UX, verified via file-wide grep evidence. PRDs updated for ATS-vs-human resume split, multi-profile V1 priority, M1B/M1C framework refinements. Audit-scope-too-narrow pattern diagnosed and corrected with structural discipline.

**Sessions shipped (5 sessions across May 20-21):**
- **Session A (May 20):** MARCELO prompt cleanup at scoreJobs + runM1 + runAutoM1. `getUserContextForPrompt()` helper introduced as canonical pattern. RESUME_TEXTS emptied. hasResume guard fixed.
- **Session C (May 21):** Read-only audit of TA + HM prep vs M1B + M1C frameworks. Surfaced 4 TA gaps + 1 HM gap + 4 cross-cutting findings including Session A escape (TA + HM still using legacy getBackground).
- **Session D (May 21):** Six framework fixes to TA + HM prep prompts. 7th MARCELO site flagged at M1D pitch.
- **Session E (May 21):** M1D pitch swap. Audit-completeness gap surfaced (3 additional live sites previously missed by enumeration-by-faith).
- **Session F (May 21):** Final sweep of 3 live sites (wsRegeneratePitch, addNotesAndPrep, buildPrepPrompt). Pre-swap + post-swap grep verification embedded in brief as structural discipline. Contamination class verifiably closed.

**Originally-scoped Session B (accordion regression fix) was diagnosed as never-built feature, not regression** — deferred to V1 polish per May 21 driver chat triage.

**PRD updates shipped May 21 (driver chat):**
- `m1b_recruiter_interview_prd.md` (NEW) — full PRD written; 6 sections (Section 7 conditional STAR removed)
- `m1c_hm_interview_prd.md` — "What does the next round look like?" added to standard inclusions; Deferred-to-V1 section added
- `resume_engine_prd.md` — Two Resume Audiences (ATS vs Human) section added; multi-profile V1 priority documented

**Dead code remaining (Day 8 cleanup, not Day 7.5 scope):**
- `runM2`, `runM3`, `runTAPrep`, `runHMPrep`, `runPitchPrep`, `updateWorkspaceChatHeader` — all confirmed dead, all consume `getBackground()`. Single Day 8 deletion pass retires all 5 sites + helper + `GENERIC_BACKGROUND_FALLBACK` (Day 9-10).


### Day 8 — Auto-search cron + signal persistence + dead code cleanup
**Goal:** Daily auto-search runs in background, emails high-scoring matches. Signals persist to Supabase. Dead-code MARCELO consumers deleted.

**Tasks:**
1. Cron job on Render (uses Render's cron jobs feature)
2. Auto-search reads users where `autosearch_enabled = true`
3. Runs each user's active search with dedup filter
4. **Auto-search filters by intent fields** (industries, target_titles, target_companies, status) — schema already supports it from Day 6
5. Sends daily email with results ≥80 score
6. **Signal persistence:** skip reasons, target companies, opportunity stages → Supabase
7. Path A inbound email handler design + parsing
8. **Dead code deletion (NEW May 21, expanded May 22):** delete `runM2`, `runM3`, `updateWorkspaceChatHeader`, `runTAPrep`, `runHMPrep`, `runPitchPrep` (Day 7.5 Session F finding). Also delete `obSkipOnboarding`, `obGoStep2`, and the `obHandleFile` `getElementById('ob-firstname'/'ob-lastname'/'ob-email'/'ob-linkedin-step1')` dead-branch lookups (Day 7 Session 2.6 findings). **Session 2.7 additions:** delete `p.email` field from `rg_profile` blob (write-only dead end after Session 2.7's email-write removal); audit `updateUserUI()` legacy `p.name` backfill (pre-Day-2 migration code, dead at closed-beta scale where all users post-May-21); decide whether Profile-page LinkedIn input surface (`profLinkedIn` at line 11116 + `saveProfileFields()`) gets removed now that Settings owns LinkedIn URL. All confirmed dead by file-wide grep. Single deletion pass eliminates 5+ `getBackground()` call sites at once, plus 3+ orphan functions, plus the cascade dead branches, plus the Session 2.7 dead fields. ~45-60 min surgical work.

9. **Logo audit across all render sites (NEW May 21):** Session 2.5 Fix #7 swapped the canonical leaf onto the auth screen but only audited that one surface. Per Marcelo May 21 review, the onboarding header logo looks "different from auth screen logo" — likely another non-canonical drift mark. Grep for all SVG logo render sites (auth screen, onboarding header, app header sidebar, anywhere else), audit each against `brand_design_system_prd.md` locked path, sync any drift to canonical. ~30 min surgical work.

10. **Email display verification (NEW May 21):** Cloudflare email obfuscation was turned off May 21 (was rewriting emails to "[email protected]" in the app). Verify across all email-rendering surfaces (Profile, Settings, recruiter contact fields, opp cards if applicable). If any surface still shows obfuscated emails, code-side bug remaining — fix. ~15 min verification, possibly some fix work.

11. **`mt_*` legacy localStorage keys audit (NEW May 22, EXPANDED May 26):** Session 2.8 smoke test discovered 5 `mt_*`-prefixed localStorage keys present on fresh `app.rolegarden.com` page load: `mt_keywords`, `mt_li_ua`, `mt_li_cookies`, `mt_apify`, `mt_v3`. These are leftover naming from the MatchTalent → Role Garden rename (May 21). Session 2.10 deleted the leak-related ones (`mt_apify`, `mt_li_cookies`, `mt_li_ua`) as part of P0 security cleanup. Remaining: `mt_v3` (active opps array), `mt_keywords` (active search keywords), and `mt_loc` (active location, surfaced Session 3) — all live load-bearing. **Session 3 added complication:** `mt_keywords` is now duplicated by `intent.target_titles` (canonical via rgData), `mt_loc` duplicated by `intent.location`. Two sources of truth for same data. Day 8 rename pass: read-old-write-new shim, migrate to `rg_opps_v3` / `rg_search_keywords` / drop `mt_loc` entirely (use `intent.location`), eventually drop the legacy read paths. Need to verify auto-search cron doesn't still consume `mt_keywords` directly before dropping. ~30-45 min surgical work.

12. **Session 2.10 cleanup surfaced (NEW May 22):**
    - **Delete dead `fetchRemoteOK` + `fetchWWR`** — both have zero callers, bodies don't touch Apify (use Remotive/Jobicy/Arbeitnow directly), already tombstoned in Session 2.10 via `_unused` param rename. Day 8 delete the function bodies entirely.
    - **Rename `fetchLinkedIn` → `fetchJSearch`** — function is the JSearch fetcher post-Day-1.5 rewire; current name is misleading. Touching it = touching JSearch path, scoped out of Session 2.10. Day 8 rename pass.
    - **Help modal staleness sweep** — references to "MatchTalent.ai" (pre-rebrand), `matchtalent_v6.html` filename, `npx serve` install instructions (no longer the deployment story), emojis (against brand rules per `_CONSTITUTION.md`). Day 8 user-facing copy sweep.
    - **`getGoogleClientId` dual-definition audit** — V8 ADDITIONS stub (line ~8955) returns empty string, overriding real `getGoogleClientId` at line 4412 that reads from localStorage. Effectively dead-paths Google OAuth Client ID feature. Could be intentional (Google export retired) but dual definition is confusing. Day 8 audit candidate.

13. **Session 3 cleanup surfaced (NEW May 26):**
    - **runM0Search admin debug panel update** — currently references "Anthropic key" / "RapidAPI key" / "Active sources" / "Time filter" / "Keywords" but NOT "Industries" / "Status" / "Location". Worth updating for closed-beta admin smoke tests so the debug panel reflects the canonical 5 inputs. ~15 min polish.
    - **Empty-intent nudge UX deleted with Your Search Profile block** — pre-Session-3 rail showed "Add target titles to improve matches." for skippers. New design has no nudge — 5 fields are right there, user sees what's empty. Decision: nudge no longer needed because surface change makes it self-evident. If post-beta data shows skippers don't engage with empty fields, revisit V1.

14. **Cross-device intent bootstrap (NEW May 26, V1 polish):** Per Session 3 build chat note: on first sign-in from a fresh browser, intent cache is empty until cloud bootstrap runs. The new bridge tolerates this (won't latch flag), but the user sees an empty rail for a moment (~100-300ms typical) until bootstrap completes. Not a closed-beta blocker. V1 polish: read intent from Supabase on `authOnLoginSuccess` BEFORE `showDiscover` runs.

15. **Pre-existing Constitution Rule #3 violation at lines 7142-7152 (NEW May 26 — Session 3.1 surfaced):** `_origSetEngageSub` override pattern. Predates Session 3 — git blame would tell when. **Exact pattern Constitution Rule #3 forbids and that Session 3 build chat caught itself doing twice.** This one was missed in earlier audits. Day 8 cleanup: identify the bottleneck the override is wrapping (probably `setEngageSub`'s sub === 'ta'/'hm'/'pitch' branching), refactor to patch the bottleneck directly. ~20-30 min surgical work.

16. **Dead-code candidates surfaced via Session 3.1 inverse audit (NEW May 26):** 45 functions with only their definition line as the only file occurrence. Most likely false positives (HTML `onclick` attrs not swept by regex). Selected genuinely-dead candidates worth Day 8 investigation:
    - `fetchRemoteOK` (line 7308) — Remote OK source dropped Day 1.5, already on Session 2.10 deletion list
    - `fetchWWR` (line 7332) — WeWorkRemotely dropped Day 1.5, already on Session 2.10 deletion list
    - `replaceM1Field` (line 3316) — pre-Day-5, investigate
    - `saveToNotepad` (line 3332) — pre-Day-5, investigate

17. **Session 3.2 cleanup surfaced (NEW May 27):**
    - **`SUPABASE_ANON_KEY` hardcoded in index.html line 3672** — legacy JWT-format publishable key. Safe in browsers (by design — publishable keys are public). But it's a literal alongside the new `sb_secret_…` server key discipline. Audit for migration to the new "publishable key" format (Supabase docs May 2026 release). Not blocking — current legacy key still works.
    - **`/api/send-email` still on shared-secret auth** — deliberate per Session 3.2 architectural decision (server-to-server use case from cron + Path A V1). When cron + Path A wire up (Day 8-9), they read `PROXY_AUTH_KEY` from their own env vars. Document explicitly. Future session may want to rename `PROXY_AUTH_KEY` → `PROXY_SERVICE_KEY` for clarity now that it's no longer a browser-shared secret.
    - **`updateKeyStatus()` setup button text** still uses the emoji "⚙" — flagged in Constitution as a hard "no emojis" rule. Pre-existing, not from Session 3.2. Worth a sweep alongside the bell emoji already on the list.
    - **`loadSupabase()` early-return branch at line 3681** uses default config without `autoRefreshToken`. Dead code in practice (nothing else loads Supabase JS), but if it ever fires the result is sessions that expire silently after 1 hour. Worth tightening: collapse the two branches into one always-with-config init.
    - **`async function authBootGate` could proactively call `getSession()` once at boot** to warm the Supabase client. Currently `claudeHeaders` does fresh `loadSupabase` on every call (cached after first). Perf note, not blocking.

18. **Accessibility warnings (NEW May 27 — Session 3.2 smoke test surfaced):** Chrome DevTools Issues panel flags 25+ violations:
    - "A form field element should have an id or name attribute" — 3 resources
    - "No label associated with a form field" — 25 resources
    Pre-existing across the codebase, not introduced by Session 3.2. Real impact: some screen readers struggle, browser autofill unreliable, Lighthouse accessibility score takes a hit. No functional break for sighted users. **Honest scope estimate:** 25 violations include some real (proper labels would help users with disabilities) AND some false positives (hidden/programmatic inputs). ~2-3h to fix all 25 where ~2h is meaningful and ~1h is busywork. **Day 8 or V1 polish.** Not closed-beta blocker.

**Estimated:** ~10-12h (heavier than prior — signal persistence from Day 6; dead code from Day 7.5 Session F; Session 2.6 orphan cleanup + logo audit + email display verification May 21; mt_* legacy keys May 22; Session 2.10 cleanup surfaced May 22; Session 3 cleanup surfaced May 26; Session 3.1 cleanup surfaced May 26 incl. `_origSetEngageSub` violation + 4 dead-code investigations; Session 3.2 cleanup surfaced May 27 incl. SUPABASE_ANON_KEY migration audit + `loadSupabase` early-return tightening + accessibility warnings sweep)

### Day 9 — OAuth (Google + Microsoft) + Buffer (NEW May 26)

**Goal:** OAuth sign-in via Google + Microsoft enabled. Closed-beta-ready auth posture for soft launch.

**Why this is Day 9 work (May 26 decision):** Marcelo correction May 26 — "closed beta" scope re-characterized as soft launch with broader audience + account continuity to V1. OAuth becomes credibility-and-friction-reduction work, not "fancy upgrade." Account continuity matters because closed-beta accounts roll forward to V1; auth method continuity prevents future migration friction.

**~~Path A handler MOVED to V1 priority (May 26).~~ Path A KILLED (May 29) — see top-of-roadmap header for full reasoning.** Both flows (auto-search delivery + user-shared JD) replaced. Replacement model: notification email + deep link (via Resend on `send.rolegarden.com`) + mobile M1A paste. No inbound email parsing infrastructure needed. `roles@rolegarden.com` subdomain never provisioned (decision: don't). The May 26 reasoning for moving Path A to V1 (mobile-share differentiation, real engineering complexity) was eclipsed by the simpler architectural read in May 29: mobile being fixed in Session 5B removes the original "email is the workaround for broken mobile" need. Auto-search cron (Day 8) remains the closed-beta "RG sends you stuff" surface, just with notification-email-not-package format.

**OAuth Tasks (Day 9 closed beta):**

**Marcelo's setup work (~1.5-2h, NOT build chat):**
1. Google Cloud Console — create OAuth 2.0 Client ID, configure authorized redirect URI to Supabase callback, get client ID + secret. ~30 min including any approval processes.
2. Microsoft Azure AD — register application, get client ID + secret, set redirect URI. ~45-60 min — Microsoft's flow is more complex than Google's.
3. Supabase dashboard — enable Google + Azure providers, paste credentials. ~15 min.

**Build chat work (~3-4h):**
1. Add "Continue with Google" + "Continue with Microsoft" buttons to auth screen UI (matching brand design system per `brand_design_system_prd.md`).
2. OAuth callback handler routing (post-success, route to onboarding if fresh user OR Search if returning).
3. Name mapping from OAuth provider to canonical localStorage keys (`given_name` → `rg_user_first_name`, `family_name` → `rg_user_last_name` for Google; `givenName` → `rg_user_first_name`, `surname` → `rg_user_last_name` for Microsoft).
4. Email mapping from OAuth provider to canonical localStorage + Supabase auth.users.
5. Edge case handling — user signed up with email/password, later tries OAuth with same email (Supabase account linking specifics).
6. Testing across both providers.

**Total OAuth scope: ~4.5-6h combined (Marcelo + build chat).**

**Apple Sign-In moved to V1 priority** — ~1-2h additional setup, Apple HIG button compliance, App Store Connect requirements. Defer.

**2FA moved to V1 priority** — production security baseline. Required when accepting payments + scaled threat surface.

**Buffer (if Day 9 finishes early):** Day 8 overflow absorption (cron / cleanup / signal persistence often runs over estimate), E2E test pass start.

**Estimated:** ~4.5-6h OAuth + buffer remainder

### Day 10 — Scoring rewrite (Option C) + JSearch upgrade + direct-employer URLs
**Goal:** Scoring engine returns rich M1A-depth on every result with streaming render. Production-ready API tier.

**Scope reset May 20:** Option C lock simplifies Day 9-10 to a single-schema rewrite (vs Option B's two-schema). Heavier prompt per call (M1A-depth), 3-jobs-per-batch (not 6), streaming render in the existing `runM0Search` loop. See `search_engine_prd.md` Result Card Structure section + workdoc card `RESULT CARD DEPTH UPGRADED Option B → Option C`.

**Tasks:**
0. **🚨 RESUME CONTEXT INTEGRITY — TWO BLOCKERS PULLED FORWARD TO SESSIONS 3.5-R + 3.5-M (NEW May 28, was "parser corruption"):** Marcelo dogfood + code trace revealed the resume-context layer is broken in two distinct ways, both pre-beta blockers, both pulled OUT of Day 10 into dedicated sessions before nav:
   - **BLOCKER 1 — PDF extraction broken (Session 3.5-R, highest priority).** `obProcessFile` PDF branch (lines ~4977-4990) uses a homegrown byte-scraper (BT...ET text-block matching + raw-ASCII fallback). Fails on FlateDecode-compressed PDFs = MOST modern PDFs (Word/Google-Docs/Canva exports). Saves raw PDF binary (`%PDF-1.7...FlateDecode...`) to `rg_resume_primary` instead of text. Confirmed via `localStorage.getItem('rg_resume_primary')` showing binary. Every assessment then reads garbage → "resume corrupted/unreadable." Affects every tester who uploads a compressed PDF. **Fix:** PDF.js shared `extractPdfText()` helper used by resume upload AND STAR import; honest-failure path (never save garbage). Marcelo-approved Option A scope: also remove the vestigial resume-variant LABEL injection from M1A prompts + UI badges (the "Player-Coach — AdTech" ghost), leaving `RESUMES`/`autoSelectResume` defined-but-unused for Day 8 deletion.
   - **BLOCKER 2 — unfinished `getUserContextForPrompt` migration (Session 3.5-M, high, pre-beta).** The Day 7.5 user-agnostic context rewrite built `getUserContextForPrompt()` (correct: reads real resume, honest fallback, never fabricates) and migrated M1A scoring + several paths (lines 6598, 6764, 7875, 10406, 10470) — but LEFT these reading fabricated persona data: M1C HM prep (line ~3401 injects `RESUMES[key].summary` — Marcelo's hardcoded "120% quota at Taboola" variant block — plus `getUserBackground()`), legacy interview prep (line ~3508 `getBackground()`), pitch prep / M1D (line ~3584 `getBackground()`), "Ask anything" chat (line ~8306 `getUserBackground()`). `getBackground`/`getUserBackground` fall back to `GENERIC_BACKGROUND_FALLBACK` (hardcoded "Senior AE, 10+ years, 110% quota" persona, line ~1878). **Effect: a closed-beta tester doing HM prep could get prep built around Marcelo's fabricated Taboola experience — a Constitution Principle 2/3 violation (no hallucinated facts) reaching testers.** **Fix:** replace all `getBackground`/`getUserBackground`/`RESUMES.summary` prompt injections with `getUserContextForPrompt()`; retire `GENERIC_BACKGROUND_FALLBACK`. Distinct, more delicate prompt-shape refactor than 3.5-R — its own session, its own grep enumeration.
   - **Sequencing:** 3.5-R (PDF) → 3.5-M (migration) → 3.5 (nav). Both blockers before nav; both well before closed beta. Day 8 then deletes the dead `RESUMES`/`autoSelectResume`/`GENERIC_BACKGROUND_FALLBACK` once nothing references them.
1. **Career profile extraction step** (per resume upload) — Haiku reads resume, returns structured profile (industries, career level, role types, themes, dealbreakers). Stored on Supabase users table. **NEW May 28: must include robust PDF parsing (PDF.js per 3.5-R) + honest-failure path. Must read CURRENT resume, not stale/deleted variant data or hardcoded personas.**
2. **M1A-depth scoring prompt** — single schema. Returns 0-100 score (native, NOT 1-10×10), 5 dimension scores, paragraph summary, 3 detailed strengths with ~2-sentence reasoning, 3 detailed gaps with ~2-sentence reasoning. Recommendation implicit in paragraph.
3. **3-jobs-per-batch** (not 6 — depth costs context).
4. **Streaming render** in `runM0Search` loop — push each batch's cards to DOM as batches complete. First batch ≤8s, all 20 cards ≤30s. Skeleton cards for unresolved batches. **Non-negotiable baseline guardrail — without streaming, Option C ships a worse first impression than today.**
5. **Retire `sc10*10` multiplier** — return native 0-100 from the prompt, remove all multiplier code paths in `renderJobResults`.
6. **Wire `getUserContextForPrompt()`** from Session A as the candidate-context input layer (already exists; was the canonical helper used across all Day 7.5 sweeps).
7. **Intent fields wired into scoring prompt** (industries, target_titles, target_companies, status).
8. **Web search infrastructure wiring (NEW May 21):**
   - Modify `claudeStream()` to accept `tools` parameter
   - Proxy changes for `web_search_20250305` tool definition
   - Activate web search in M1B Section 4 (Company Insights Worth Knowing — prompt language already forward-compat from Session D)
   - Activate web search in M1C "The Interviewer" + strategic context sections
   - Make web search available to M1D pitch prep (current 5-section interim ships with tools enabled)
   - Cost: ~$0.01-0.03 per prep with web search; closed-beta total negligible
   - **Why this is in Day 9-10, not deferred to V1:** Marcelo's May 21 reflection that web search is structurally load-bearing for the prep product. Originally framed as a deferrable feature; correctly diagnosed as architectural for M1B/M1C/M1D quality.
9. **Retire `GENERIC_BACKGROUND_FALLBACK` constant** — after Day 8 dead-code deletion lands, the constant has zero consumers. Delete.
10. **Retire `getBackground()` helper** — same as #9. Zero consumers after Day 8 deletion.
11. **Direct-employer URL preference logic (NEW May 22 — legal/UX driven):**
    - JSearch returns `apply_options` array per job — multiple apply URLs including direct employer ATS (Greenhouse, Lever, Workday, Ashby, employer-careers-page) AND aggregator URLs (LinkedIn, Indeed, Glassdoor).
    - Pre-process: parse `apply_options`, prefer direct-employer URLs over LinkedIn/aggregator URLs for the result-card primary Apply CTA.
    - Detection heuristic: URL hostname contains `greenhouse.io`, `lever.co`, `workday.com`, `ashbyhq.com`, `myworkdayjobs.com`, OR matches the company's name in the careers subdomain (`careers.{company}.com`, `jobs.{company}.com`).
    - Fall back to LinkedIn/aggregator URL only when no direct-employer URL is available.
    - De-emphasize "via LinkedIn" source attribution in card chrome when a direct-employer URL is used (source still shown in card detail view, just not the headline).
    - **Why this is here, not deferred:** real product improvement (applying direct to employer is generally better UX — recruiters track LinkedIn-applies separately, direct-apply sometimes signals higher intent) AND meaningful legal-posture improvement (reduces "RG displays LinkedIn-sourced content prominently in a commercial product" surface). Driver-chat call May 22 after legal-risk reflection.
    - **Estimated scope:** ~1-2h additional within Day 10 work. Pre-processing logic in scoring pipeline + render-time URL preference in result card.
12. Eyeball test post-ship: 5 searches across Marcelo's profile range, validates rubric quality. Reality (a) defensible / (b) tighten / (c) rewrite — decision post-ship.
13. Upgrade JSearch to Pro tier $25/mo.
14. LinkedIn coverage validation via Marcelo dogfood (10 jobs test).

**Estimated:** ~12-14h (was ~10-12h; +1-2h for direct-employer URL preference logic per May 22 driver-chat decision on legal posture + UX improvement). Probably 4 sessions.

**Fallback lever:** if testers report search still feels slow after streaming ships, cap initial render at 10 cards with "Load 10 more" button. Not in initial Day 9-10 scope — added only if streaming proves insufficient.

**Cost reset:** per-user/month bulk scoring jumps from PRD's prior $0.10 estimate to ~$0.50 (5x — M1A-depth on every card). Closed-beta total ~$5/mo across 5-7 testers. V1 pricing tier absorbs the cost. Web search adds ~$0.10-0.30/user/mo at closed-beta usage — also absorbed.

### Days 11-12 — Buffer + testing
**Goal:** End-to-end testing. Fix what's broken.

**Tasks:**
1. Full sign-up → onboarding → search → opp → outreach → prep flow test on `app.rolegarden.com`
2. Path A end-to-end (forward email → Email 1 → click → Email 2)
3. Auto-search cron verified
4. Cross-device test (sign in on phone after laptop)
5. Bug fixes
6. (Production deploy already shipped Day 7 Session 2 — buffer days are pure testing/fixes now, not infrastructure)

**Estimated:** ~5-6h/day

### Day 13 — Soft launch
**Goal:** First 2-3 invited testers.

**Tasks:**
1. Onboard 2-3 closest friends as testers
2. Watch first sessions live (over Zoom if possible)
3. Capture every bug + every "what just happened" moment
4. Same-day patches for critical issues

### Day 14 — Closed beta opens
**Goal:** Expand to 5-7 testers total.

**Tasks:**
1. Onboard remaining testers
2. Set up feedback capture (in-app widget or simple email)
3. Daily check-in for first week

---

## Post-Beta (Day 15+)

**Validation period (Days 13-21):**
- Measure reply rate via Path A (target: ≥15% within 7 days)
- Self-reported comparison ("better than usual approach")
- Capture bugs + UX friction
- Day 21 decision: go/no-go on reply-rate-as-positioning

**Post-beta priorities (Day 22+ — toward V1 public launch):**

1. **Chat-driven onboarding rebuild (V1)** (~6-10h)
   - RG asks contextual questions, not form fields
   - Adaptive flow (career pivot path differs from continuation path)
   - Captures intent (industries, target titles, target companies, status) through dialog
   - **Auto-populates the first search fields** from chat capture — user lands on a personalized first search, not an empty form
   - Replaces the Day 6 form-based interim

2. **AI-first chat-driven UX rebuild — M1A/B/C/D (V1)** (~12-18h focused work)
   - Build reusable chat workflow helper
   - Convert M1A/B/C/D + post-call notes flows to chat-driven
   - This is the broader rebuild; onboarding (#1) is separate scope

3. **M1D Strategy Coach** (~15-20h)
   - Multi-turn coaching following 7-element Marcelo Framework
   - Web search for prospect research
   - Progressive strategy doc + multi-format artifacts

4. **Phase 2 search learning signals — REQUIRED for V1** (~6-8h)
   - Wire captured signals INTO scoring prompt (skip reasons, opp creations, save events)
   - Test against historical data
   - **Now V1-mandatory because closed-beta marketing copy commits to "Role Garden learns from every job you save, skip, or apply to."** Truthful-future framing accepted at closed beta with explicit gap window; V1 must deliver. See `search_engine_prd.md` Marketing Framing section.

5. **Multi-profile architecture — V1 finish (~4-6h, was ~8-12h)** — UPDATED May 22
   - **Closed-beta foundation lands in Day 7 Sessions 3-5** (search-rail rework) — saved-searches infrastructure becomes the multi-profile mechanism. Each "My Search" entry is essentially a profile.
   - **V1 addition:** add `resume_id` field to saved-search entries. UI lets user select which resume to score against per search.
   - **Closed beta has ONE resume per user.** V1 enables multiple resumes + per-search-resume selection.
   - Scoring code reads from `getUserContextForPrompt()` — V1 extends helper to accept `resume_id` parameter
   - **Why scope shrank from 8-12h → 4-6h:** the rail rework lands the hard architectural work (saved searches with their own inputs). V1 only adds the resume-per-search layer.

6. **Two-resume engine: human-readable version (V1)** (~4-6h) — NEW May 20 from friend feedback
   - ATS/system version ships Day 7 for closed beta
   - Human-readable narrative version (visually structured, narrative-led) is post-beta
   - Both versions per role — user picks which to apply with based on platform

7. **Skills section in M1 framework (V1.5)** — NEW May 20 from friend feedback
   - Pairs with ATS resume work — skills are what ATS screens on
   - "Skills Match" sub-panel in M1 assessment feeds tailored resume + recruiter prep ("they'll ask about X — here's your STAR")

8. **Button rename pass (post-beta CSS sweep)** — NEW May 20 from friend feedback
   - "Create Opportunity" / "My Opportunities" both betray Marcelo's CRM background
   - Deliberate naming round, not impulse rename (candidates: "Track Role" / "My Pipeline" / "My Applications")

9. **Targeted Companies as scoring signal** (V1.5)

10. **Public V1: Stripe + 14-day trial + marketing site at `rolegarden.com` apex (Framer)** — domain architecture locked May 21. Marketing site uses Framer's visual builder + CMS for blog and paid-media landing pages. Apex serves marketing; app stays at `app.rolegarden.com`. Hero structure already drafted in workdoc Homepage section (Find / Learn / Prep three-moment pattern; tagline "Your career, cultivated."; positioning Option C for closed beta, Option B leaning for V1 — workdoc-recommended Option A or B locked at V1 launch).

11. **~~Path A handler — mobile-share email-in~~** **KILLED (May 29).** Was V1 priority (moved from Day 9 on May 26); fully retired May 29. Reasoning: closed-beta Session 5B ships limited mobile (Search + My Opportunities + M1A) which removes the original "mobile is broken, email is the workaround" need entirely. Two flows replaced cleanly: (a) auto-search delivery → notification email + deep link (LinkedIn/Indeed pattern, less infra, no inbound parsing, no per-email M1A cost, state stays in app); (b) user-shared JD → mobile M1A paste-and-go (strictly fewer steps than forward-and-wait flow). Removes: Resend inbound webhook setup, JD parsing prompt, email-back template, multi-message reply handling, `roles@rolegarden.com` subdomain provisioning, "Try mobile share" CTA on Search empty state. Adds: simple notification email build (~1-2h, low complexity) reusing existing Resend outbound. **Net effect: less infrastructure, simpler architecture, better UX, lower Anthropic spend.** "Try mobile share" surface stays hidden permanently — no V1 unhiding planned. Decision documented in detail in `2026-05-29-day8-email-architecture-workspace-migration.md` session summary.

12. **Apple Sign-In OAuth (V1, NEW May 26)** — OAuth provider deferred from closed beta. Apple Sign-In has additional setup complexity (App Store Connect requirements, certificates, button design must follow Apple HIG strict size/color/text rules), ~1-2h additional time investment vs Google/Microsoft. Real value at V1 when iOS user proportion matters more.

13. **2FA on login (V1, NEW May 26)** — Two-factor authentication for production security baseline. Closed beta ships with email confirmation only (one-time at signup, not 2FA). V1 adds opt-in 2FA: TOTP authenticator app (Google Authenticator, Authy) preferred over email-based 2FA for stronger threat model. Required when accepting payment + scaled threat surface. Closed beta defers because 4-step-login UX (signup → confirm → login → 2FA → app) creates real friction barrier for soft-launch adoption signal; threat model at 5-50 invited testers doesn't justify it.

14. **Enriched Profile Context Capture (V1, NEW May 26)** — ~5-7h. Power users + AI-fluent users want to add context beyond resume to strengthen scoring/results. Architectural decision deferred from closed beta to avoid stacking pivots. Two real components:
    - **(a) Wire STAR library into scoring prompt** — STAR stories already captured during HM prep workspace work ARE structured behavioral context (situation/task/action/result). System captures rich context; just doesn't apply it to scoring yet. Smaller scope, ~2-3h. May be enough on its own.
    - **(b) Optional free-form profile context capture** — chat-driven surface OR persistent textarea on My Resumes page. User types/uploads supplementary context (presentations, case studies, "I'm pivoting from X to Y" framing, constraints). Honors Constitution Principle 1 (chat-driven UX). ~3-4h.
    - **Validation step before building:** ask closed-beta testers in feedback interviews — "Would adding context beyond resume help your matches?" Real signal vs assumption. Source of original feedback: new voice who hasn't used the product (weights lower than tester signal).
    - **Implementation order:** ship (a) first — leverages existing STAR data, smaller scope. Decide on (b) based on closed-beta tester feedback signal.

15. **Full mobile responsive CSS (V1, NEW May 26 evening)** — ~6-10h. Closed beta ships with mobile gate ("desktop-only, we'll email you when mobile launches") at Session 3.4. V1 invests properly in mobile responsive layout. Real work involves: rail-output two-column → single-column stack on phones; resume upload UI sized for mobile viewport; M-tab workspaces redesigned for phone (probably accordion-only, no side-by-side framework + content); modals sized for mobile; touch-target sizing; iOS Safari quirks. Path A (V1 priority #11) depends on mobile working — "you forwarded a job, now check the match" wants mobile. Real V1 expansion of usable surface.

16. **LLC formation + business banking + business address (V1 prerequisite, NEW May 26 evening)** — Formally logged. Real V1 prerequisites before paid tier launches:
    - **Florida LLC formation via Sunbiz.org DIY** — ~30 min filing, $125 one-time. File ~2-3 weeks before V1 launch.
    - **EIN from IRS.gov** — ~10 min, free, requires LLC formed first.
    - **Operating Agreement** — single-member template from Rocket Lawyer / LegalZoom, ~1 hour, free.
    - **BoA Business Advantage Fundamentals** — at Marcelo's Platinum Preferred Rewards tier (>$50k combined BoA personal + Merrill assets), monthly $16 fee waived. Open via branch appointment with Merrill relationship referenced. ~1 hour.
    - **BoA Business Advantage Customized Cash Rewards card** — apply ~2 weeks after account opens. Choose "online services" 3% category × 1.5 Platinum bonus = 4.5% on SaaS infra spend.
    - **Business address strategy (CORRECTED May 27 evening — earlier pricing assumptions were wrong):** Three real options ranked by value:
      - **(A) Northwest Registered Agent Virtual Office: $29/mo** — bundles unique commercial building address + suite number + real office lease (passes bank verification) + unlimited mail forwarding + dedicated VoIP phone line included + month-to-month. **Pairs naturally with Northwest RA service ($125/yr) — single provider, single dashboard. Total ~$40/mo all-in.** Recommended path IF Northwest has a Florida virtual office in Orlando metro (verify before signup — visit `northwestregisteredagent.com/business-address/virtual-office/florida`). Phone line included addresses credibility concern.
      - **(B) Northwest RA + Opus Virtual Offices downtown Orlando ($99/mo at 200 E Robinson Street, Lake Eola area)** — Class A building, transparent pricing, bundled live receptionist, month-to-month. Better location than Northwest's Florida office (if not Orlando) but $70/mo more expensive than option A. Total ~$108/mo all-in.
      - **(C) Northwest RA + The Worx ($49/mo + $25/mo phone = $74/mo, Winegard Road south Orlando)** — Orlando-located but not downtown, $40 setup fee, bilingual front desk. Total ~$83/mo all-in.
      - **iWorkspaces was $99/mo NOT $59/mo** — earlier roadmap pricing was incorrect, Marcelo flagged the discrepancy. Removed as recommended option.
      - **Phone-line rationale:** Stripe + business bank verifications sometimes require phone; user trust signal in closed beta; separation from personal. Even with voicemail-only setup ("recording-assisted"), credibility benefit is real.
      - **Discipline checkpoint:** upgrade business address BEFORE V1 marketing surfaces go live. Closed beta can technically launch with home address as Principal + Northwest as RA ($0 monthly cost) — defer the virtual office until soft-launch testers actually need a public business address (ToS publication, Stripe receipts, marketing site).
      - **Decision deferred May 27 evening pending Marcelo research:** verify Northwest Florida virtual office location + confirm phone-line specs.
    - **CPA consult re: sales tax + multi-state nexus** — ~1h, $200-400, before V1 launch.
    - **Total cost:** $125 LLC + $0 EIN + $0 OA + $0/mo BoA + optional $15-150/mo address. First-month: ~$125-275 total, ~$0-150/mo ongoing.

17. **Hosting infrastructure migration plan (V1+, NEW May 26 evening)** — Netlify currently free tier at 300 credits/month. Marcelo discovered "Low on credits — 149 remaining" mid-May. **Near-term: upgrade to Netlify Personal ($9/mo, 1,000 credits) BEFORE closed beta launch** to prevent deploy blocks during launch week. **V1 post-launch: evaluate Cloudflare Pages migration** — Cloudflare DNS already manages `rolegarden.com` (existing relationship). At 10K users estimated: Netlify ~$163/mo overage credits, Cloudflare Pages likely $0-5/mo (free tier covers static-heavy sites). ~1-2h migration effort, do AFTER V1 stabilizes (not during sprint).

---

## V1 Pre-Launch Checklist (logged May 21 — items to ship BEFORE public V1 launch)

These are items surfaced during closed-beta sprint work that don't block beta but MUST land before V1 audience is exposed:

### Trust / Legal / Compliance
- [ ] **Real Terms of Service** — hosted at `rolegarden.com/terms` or `app.rolegarden.com/terms`. Closed beta uses soft framing ("By creating an account, you agree to use Role Garden during closed beta…"). V1 needs real ToS with proper checkbox.
- [ ] **Real Privacy Policy** — same hosting pattern. References data handling, retention, third-party (Supabase, Resend, JSearch).
- [ ] **ToS + Privacy checkbox on signup** — replaces current soft framing on Create Account form.
- [ ] **Marketing checkbox on signup** — separate from ToS, CAN-SPAM compliance: "Send me product updates and tips." Off by default.
- [ ] **Legal consult with tech attorney** (NEW May 22 — $500-1500, ~1h) — pre-V1-launch. Specifically discuss: (a) RG's posture as a JSearch customer displaying LinkedIn-sourced job postings — is there exposure to LinkedIn legal action? (b) Should anything be formalized with JSearch about their data sourcing posture? (c) Marketing claims about LinkedIn coverage — what's safe to say vs avoid? (d) ToS / Privacy Policy review. Not needed for closed beta (5-7 personal invitees, no public marketing). Required for V1.
- [ ] **Marketing copy review for "we search LinkedIn" claims** (NEW May 22) — replace anywhere in the workdoc, PRDs, marketing site, app UI with "we search the web for jobs that match" (accurate — JSearch is a Google-jobs aggregator that includes LinkedIn, Indeed, Glassdoor, employer pages, and more). NOT "we search LinkedIn." Subtle but real positioning shift that reduces both legal exposure and dependence on a single source name.
- [ ] **JSearch ToS review for commercial display** (NEW May 22) — confirm JSearch's terms of service let RG display their data commercially. They should — that's literally what they sell — but worth a documented check before V1 marketing claims compound.

### Bot / Security
- [ ] **reCAPTCHA v3 on signup** — ~15 min Google reCAPTCHA setup + script tag + verify-on-backend. Not needed at closed beta (invite-only); needed at V1 (public URL exposure).
- [ ] **`robots.txt` on `app.rolegarden.com`** — block all crawlers (`User-agent: * Disallow: /`). Standalone task post-Session-2.5. App must stay un-indexed.
- [ ] **HARD RULE: never scrape LinkedIn directly** (NEW May 22) — RG stays on the customer-of-aggregator side of the legal line (JSearch is the scraper, RG is the downstream consumer). The moment RG or any contractor writes a LinkedIn scraper, RG enters the hiQ-v-LinkedIn risk class. Documented here so future-Marcelo doesn't get talked into it without understanding the precedent.
- [ ] **Hardcoded literals audit before every credential-touching session** (NEW May 22 — Session 2.10 incidental P0 finding) — Session 2.9 surfaced that lines 8855-8857 of index.html contained Marcelo's real production Apify API token + LinkedIn `li_at`/`JSESSIONID`/`bscookie` cookies, hardcoded into bundled JS and shipped to every visitor. Live P0 exposure. Session 2.10 deleted the entire Apify path (17 sites, 32 matches). **Future sessions adding ANY infrastructure credentials must include a pre-edit grep for credential patterns: `apify_api_`, `sk-ant-`, `eyJ` (JWT prefix), `li_at=`, `JSESSIONID`, `SUPABASE_`, any obvious token pattern.** Add this to `_CONSTITUTION.md` Pre-Launch Checklist as a standing discipline. The Day 7.5 grep-discipline pattern extends here with credential-pattern targets vs prompt-language targets.

### Auth UX
- [ ] **Split-screen auth redesign** — value prop framing on left, signup form on right. Standard SaaS pattern (Linear, Notion, Stripe). Sources visuals from marketing site, so paired with Framer build.
- [ ] **Onboarding log-out confirmation modal** — closed beta ships bare link, V1 polish should add "Exit setup? Your account will be saved but you'll need to complete onboarding next time" confirmation.
- [ ] **Apple Sign-In OAuth** (NEW May 26) — closed beta ships Google + Microsoft OAuth (Day 9). Apple deferred to V1: requires App Store Connect setup, certificates, button strictly per Apple HIG (size/color/text rules). ~1-2h additional vs Google/Microsoft setup. Add when iOS coverage matters more (V1 marketing).
- [ ] **2FA on login** (NEW May 26) — closed beta has email confirmation only (signup), no 2FA on login. V1: opt-in 2FA via TOTP (Google Authenticator / Authy) preferred over email-based. Required when accepting payment + scaled threat surface. ~3-4h. Closed beta defers because 4-step-login UX (signup → confirm → login → 2FA → app) creates real friction barrier for soft-launch adoption signal; threat model at 5-50 invited testers doesn't justify it.

### Brand
- [ ] **Leaf logo identity consolidation (V1 brand work, NEW May 27)** — App currently uses multiple leaf treatments with no consistent brand identity: small header logo (geometric SVG path optimized for sidebar), 64px welcome 🌱 emoji (output area on empty Search), Ready step 🌱 emoji (orphan after Session 3.4 removal), `rg-loading-leaf` SVG animation (loading states — **NEW May 28: Marcelo confirmed the loading leaf shown during M1A assessment differs from the welcome leaf; loading states should use ONE canonical animated leaf across ALL loading surfaces — assessment, search, M-tab generation**). Real V1 brand work: (a) design ONE canonical logo with multi-scale proportions [sidebar 16-24px / header 32-40px / hero 64-128px], (b) design a SEPARATE animated leaf for hero/loading moments [welcome, loading, success states] — different purpose, different treatment, used consistently everywhere a loading or hero leaf appears, (c) replace ALL 🌱 emoji uses with the canonical SVG once design lands. Constitution emoji exception for brand mascot 🌱 is interim — V1 brand work removes the exception entirely. Estimated: ~4-6h design + ~2h implementation sweep.
- [ ] **Mobile-responsive web nav** — current app is desktop-first. Closed-beta path is Path A (mobile email forwarding). V1 needs proper responsive nav for the app.
- [ ] **All auth form labels DM Mono uppercase per brand_design_system_prd** (NEW May 21) — Session 2.6 Fix #1 pulled First/Last labels from DM Mono → Lexend inherited to match Email + Password (visual mismatch fix). PRD actually specifies DM Mono for eyebrow labels — so the right V1 fix is the inverse: pull Email + Password labels TO DM Mono so all 4 match the PRD pattern.
- [ ] **Onboarding DOM ID rename** (NEW May 21) — Session 2.6 dropped About You step but kept DOM IDs `ob-step-2/3/4` for minimal collateral. Visible step numbers (1/2/3) now differ from IDs. V1 polish to rename IDs cleanly. Three in-code comments mark this.

### Social Media / Paid Marketing Infrastructure (NEW May 29-30)
**Pages live (created May 29-30 in separate setup chat — `2026-05-29-day8-social-pages-setup.md`):**
- ✅ Facebook Page: "Role Garden", categories Software Company → Jobs & Occupations → Career Counselor, CTA Sign Up → app.rolegarden.com, contact email support@rolegarden.com
- ✅ LinkedIn Company Page: "Role Garden", Software Development industry, size 1, privately held
- ⚠️ Instagram: account created, NOT yet connected to FB Business Suite
- ✅ Brand assets: 8 files generated (circle 512px PNG for FB+Workspace, square 512px PNG for LinkedIn, FB cover 820×360, LinkedIn cover 1128×191, all with SVG sources + 2× retina). All use canonical leaf path + #1B6B3A brand green + #f5f7f5 background. Stored locally — file under `brand-assets/`.
- ✅ Workspace profile photo synced via admin path

**Honest residual from setup session:**
- [ ] **Long FB About section entry** — 5-step wizard only captured short Bio (101-char). Long version drafted but needs entry via Page settings → About → Description. Final text in social-pages summary.
- [ ] **FB cover image positioning check** — confirm crop in FB editor on upload
- [ ] **Connect IG → FB Business Suite** (required for cross-platform paid Meta ads)
- [ ] **Documented target audience definition gap** — driver-chat fabricated "Built for senior individual contributors and mid-career professionals" in first About draft; Marcelo caught it ("where did you get that from?"). No PRD-source persona exists. Required before Meta ad targeting; pulls demographics, interests, behaviors out of "tired of job boards" rhetoric. Open V1 question.

**Meta paid infrastructure (NOT touched in setup session, parked for V1 paid activation):**
- [ ] **Business Suite / Business Portfolio setup** — business.facebook.com → create with marcelo.m@rolegarden.com, business legal name "Role Garden LLC" (post-Sunbiz approval — currently pending 2-5 day clock), FL address, business phone
- [ ] **Connect Page to Business Portfolio** (moves Page ownership from personal FB → business entity)
- [ ] **Create Ad Account** — USD, ET timezone, business card billing (BoA Customized Cash Rewards card once it lands, ~2 weeks post-BoA account opening)
- [ ] **Domain verification for Meta** — add `rolegarden.com` via DNS TXT (Cloudflare). Required-ish for iOS 14.5+ conversion tracking.
- [ ] **Pixel install** — base snippet in `index.html` `<head>`. `CompleteRegistration` event in signup success branch (after Supabase user confirmed-created, NOT on form submit — false positives on duplicate-email signups). Touches `index.html` → apply Constitution browser-load verification discipline.
- [ ] **Conversions API (server-side)** — through Render proxy. Optional for closed beta, required-ish before V1 paid spend.
- [ ] **2FA on all admin surfaces** — personal FB, Business Portfolio admin, Workspace admin. Non-negotiable before paid spend.

**Organic floor for Pages (4-5 posts before turning on paid, V1-prep work):**
- [ ] Founding post (pinned)
- [ ] Product moment (one screenshot, score card or My Opportunities)
- [ ] Anti-pattern post (counter-positioning, voice-matched)
- [ ] Founder note (humanizes the page)
- [ ] One useful thing (job-search tip with product POV)

**Meta Verified subscription:** Deferred. ~$15-22/mo, requires minimum account tenure (just-filed LLC + just-created Page fails check), doesn't unlock ads. Revisit V1 post-beta when 30+ days of activity exists.

**Brand-treatment-inconsistency flag (from social setup session):** Currently shipping circle treatment (FB + Workspace) AND square treatment (LinkedIn) in parallel. Both use same canonical leaf + brand green + white. Defensible per platform (FB always crops circular; LinkedIn renders square in many contexts). Doubles the brand-treatment-variance problem already flagged in `Leaf logo identity consolidation` above. **All three+ social surfaces should swap in same 24h window when V1 canonical logo lands** — staggered swaps read as "is this the same company?", coordinated swap reads as deliberate rebrand.

### Product / UX (post-beta polish)
- [ ] **Follow-up email: kill the button, move to chat-driven generation (NEW May 30, post-3.7.3)** — current model has a "Draft my follow-up email" click target in the output area below prep, which generates from the notes pool. Marcelo's instinct (correct): the button-based generation is the architectural outlier — every other prep generation (M1B/C/D + frameworkless prep) is chat-driven. The button assumes who you're emailing, what kind of follow-up (thank-you / check-in / different angle), what to emphasize, what to skip. A chat ask handles all of that with real context: "draft a thank-you to the recruiter focused on the team-fit conversation." Consistent with Principle 1 (decisions in chat). Removes the brittle "what if there are no notes yet" UX edge case. Discoverability solved with chat seed suggestions. Counter-argument is mild (one more thing user thinks to ask vs click). 3.7.3 fixes the click-bug for closed beta; this redesign is V1 polish. Touches: rgDraftFollowup deletion, the renderFollowupEmailCard click target removal, chat hint additions for follow-up discoverability. Est ~2-3h.


### Contingency planning + dispute handling (NEW May 27 — logged to revisit pre-V1)
Marcelo's framing: "what can go wrong with users and how to handle it with minimum human interaction." Initial scope notes, revisit pre-V1 to expand into concrete implementation plan.

**Real dispute categories to plan for:**
- **Billing disputes** — forgot to cancel before trial converted, "I didn't use it" retroactive refund requests, subscription tier confusion, cancellation didn't work
- **Product quality disputes** — "scores aren't accurate," "AI gave bad prep advice," "resume looks worse after tailoring," "I didn't get any interviews"
- **Data + privacy disputes** — delete my data (GDPR/CCPA right-to-delete), who has my resume, account compromise / unauthorized access
- **Technical disputes** — lost access to signup email, OAuth account locked, mobile broken, feature broken for me
- **Edge cases** — underage user got past terms, public figure data concerns, AI hallucination in user-facing content

**Industry-standard mitigations for B2C SaaS ~$5-30/mo tier:**
- **Stripe Customer Portal** for self-service billing (cancel, plan changes, invoices) — $0 add-on
- **Stripe Chargeback Protection** — defends bank-filed disputes — free with standard Stripe
- **30-day money-back policy** for documented product issues + cancel-anytime (no refund for current period) is common pattern
- **`support@rolegarden.com` shared inbox**, 24h response target
- **Self-service account deletion + data export** (GDPR-required for some; good practice for all)
- **AI/FAQ widget** for common questions — could be built using RG's own AI infrastructure (meta + cheap)
- **Logged prompt+output for every M-tab generation** — when user disputes "AI told me X," you have records

**RG-specific risks:**
- **AI-content disputes** (generated outreach / prep / resume tailoring can go wrong) — mitigate with ToS disclaimer + "Regenerate this section" button + content-review framing ("review before sending")
- **Resume tailoring outcomes** — never frame as "this WILL get interviews"; frame as "starting point, review before sending"
- **Outcome disputes** (job search is multi-variable) — ToS must explicitly say RG can't promise outcomes

**V1 launch baseline (real cost ~1-2 days setup + $0-50/mo tooling):**
- Clear ToS + refund policy + AI content disclaimer (legal consult covers all of these)
- Stripe Customer Portal + Chargeback Protection
- Self-service account deletion + data export
- `support@rolegarden.com` provisioned
- AI/FAQ widget for common questions
- M-tab generation logging (prompt + output) for dispute resolution

**The legal consult is the keystone** — covers ToS + refund policy + AI disclaimer + outcome disclaimer + privacy. Already logged as V1 prerequisite ($500-1500). Single consult prevents most disputes from being expensive. Don't skip. Pre-V1 task to expand this section into concrete implementation plan with tooling choices, copy drafts, and self-service flow specs.

### SEO + AEO
- [ ] **`seo_aeo_strategy_prd.md` expansion** — currently a stub (created May 21). Expands to full strategy when positioning is locked and Framer marketing site build starts.
- [ ] **`Organization` + `WebApplication` JSON-LD structured data** on marketing site — critical for AEO. AI engines (ChatGPT, Perplexity, Claude search) use this to decide what to cite.
- [ ] **Title tags + meta descriptions** per marketing site page (Framer per-page settings).
- [ ] **OpenGraph + Twitter card meta** per marketing site page.
- [ ] **Substantive FAQ on marketing site** — specific questions ("How does Role Garden's scoring engine differ from LinkedIn Easy Apply?"), not generic. Cited by AI answer engines.
- [ ] **Sitemap.xml** — Framer auto-generates.

### Positioning
- [ ] **Lock positioning Option A vs B** — workdoc has both ranked. Decision after 2-3 closed-beta testers have used the product. May 21 leaning: Option B ("Outreach. Match. Prep. Apply. — All in one place."). See workdoc Homepage section for full options.

### Product
- [ ] **Phone number capture** — never. Logged here so future-Marcelo doesn't get talked into it without a real use case. Add when SMS feature exists.
- [ ] **3-5 cornerstone marketing pages drafted** (homepage + How RG Works + audience-specific + pricing + FAQ).
- [ ] **LinkedIn URL — V1 networking feature trigger** (NEW May 21) — Session 2.6 moved LinkedIn URL field to Settings → Account. Currently optional, captured via `profile.linkedIn`. When V1 networking features ship ("find mutual connections at this company"), prompt users to add LinkedIn URL with a real use case rather than asking at signup.
- [ ] **Resume Haiku extraction name conflict resolution** (NEW May 21) — Session 2.6 deferred. If a tester uploads a resume with name different from what they typed at signup, signup-form name wins (correct intuition). V1 decide: should resume extraction ever override signup-form name? Current behavior: no.
- [ ] **Supabase user_metadata as canonical source for names + email** (NEW May 22) — Session 2.7 added one-way sync (metadata → localStorage on login). V1: if user can change name in Settings AND propagate back to Supabase auth, that's a `supabase.auth.updateUser({data:{first_name,last_name}})` call inside `saveUserSettings`. Closed beta acceptable without (testers won't be changing names mid-flight).
- [ ] **Profile-page LinkedIn input surface** (NEW May 22) — `profLinkedIn` at line ~11116 + `saveProfileFields()` are still wired. Same `p.linkedIn` field as Settings → Account, not broken, but duplicate surface. V1 decide: remove Profile-page input OR keep both surfaces and document why. (Day 8 cleanup candidate alternatively.)
- [ ] **All app labels use one convention** (NEW May 22) — currently auth uses Lexend uppercase eyebrow (Session 2.5 Fix #2 / Session 2.6 Fix #1), Settings uses Lexend sentence-case 11px (Session 2.7 Fix #1). Pick one convention and apply across. Already partially logged under Brand → "All auth form labels DM Mono"; expands to "all labels."

---

18. **Workday-style resume builder fallback** (V1.5 — onboarding-friction reduction, NEW May 26)
    - **Use case:** users without a ready resume (career changers, recent grads, returnship users, people who haven't updated in years) can't onboard cleanly today. Resume upload is the gate to entry by design (RG's value depends on resume context for scoring).
    - **V1.5 solution:** Workday-style structured form as fallback when user clicks "I don't have a resume ready" OR similar opt-in path. Fields: name, current/most-recent role, years of experience, key skills, brief summary, education. RG generates a basic resume from the structured input, treats it as the user's resume for scoring purposes.
    - **Why V1.5 not closed beta:** closed beta scope discipline. Audience for closed beta = professional job seekers with resumes ready. Resume-less audience (returnship, career-change) is V1.5 expansion market.
    - **Real moat:** lowers friction for users industry-leaders ignore (LinkedIn assumes everyone has a profile). Expands TAM in V1.5 without adding LLM cost.
    - **Source of insight (May 26):** Marcelo flagged during architectural pivot discussion — "we are not a resume builder tool. Or we can build a resume builder v2 with fields like Workday as a fallback."
    - **Estimated:** ~6-8h. Form design + storage + resume-from-structured-input generation (could be Haiku template) + integration with existing resume slot in user model.

19. **Always-On Plan tier** (V1.5 — career-cultivation pricing)

20. **Full rail unification — LinkedIn-style filter bar** (V1.5 — UX polish, not debt)
   - Collapse current TITLES + Location + intent fields (industries, target_companies, status) into a single LinkedIn-style filter bar with count badges (e.g., "Industries 3 ▼")
   - All filters collapsed by default; click chip to expand inline
   - Replaces the closed-beta mixed pattern (TITLES + Location expanded inputs, intent fields as a separate collapsible block from Session 5)
   - **Honest framing — this is polish, not debt.** Closed-beta launch state (intent in rail block from Session 5 + current TITLES + Location separate + rail-TITLES pre-populate bridge from intent) functions correctly. This V1.5 work upgrades the visual model; nothing is broken without it
   - Trigger to promote to V1 priority: closed-beta tester feedback that the mixed pattern (separate intent block vs inline rail inputs) is confusing
   - Estimated ~5-6h focused work

---

## Closed Beta Blockers (Must Ship)

These are non-negotiable before launch:

1. **Search Relevance + Scoring Rewrite** — Day 9-10
2. ✅ **Onboarding Intent Capture (form-based interim)** — Day 6 Session 4 shipped
3. ✅ **Historical Search Dedup** — Day 6 Session 3 shipped (suppression filter against opps + skipped jobs)
4. ✅ **Save for Later** — Day 6 Session 2 shipped (3-button card footer + dedicated rail view)
5. **Supabase data sync** — Day 6-7 (write-through `rgData` shipped Session 1; legacy localStorage-read replacement still pending — Day 7+ infrastructure work)
6. **Tailored Resume PDF (ATS/system version)** — Day 7 Sessions 6-8 (shifted from 3-5 on May 22 to accommodate search-rail rework)
7. ✅ **Resend Integration** — Day 7 Session 1 shipped (May 18-20)
8. **Email confirmation re-enabled** — Day 7 Session 2 (gated on Site URL fix)
9. **Auto-search cron (intent-filtered)** — Day 8
10. **Path A email handler** — Day 9
11. **JSearch Pro tier** — Day 10
12. ✅ **Production URL (Netlify at `app.rolegarden.com`)** — Day 7 Session 2 shipped (May 21). Pulled forward from Day 11-12 buffer. Domain architecture locked: app on subdomain, apex reserved for V1 marketing site.
13. ✅ **Intent visible in search rail** — Day 6 Session 5 shipped
14. ✅ **MARCELO contamination cleanup** — Day 7.5 Sessions A/D/E/F shipped (May 20-21). All 9 live-reachable prompt sites verified clean via file-wide grep. 5 dead-code consumers remain for Day 8 deletion.
15. ❌→⏸ **Accordion regression fix** — diagnosed as never-built feature (not regression). Deferred to V1 polish per May 21 driver chat triage. Current rendering is functional single-accordion-with-markdown — structure improvement is V1 work.
16. **Web search infrastructure wiring** — Day 9-10 (NEW May 21). Required for M1B Section 4 (Company Insights Worth Knowing) + M1C "The Interviewer" + strategic context + M1D pitch availability. Originally framed as deferrable to V1; correctly diagnosed May 21 as structurally load-bearing for prep product quality.

---

## Deferred to Post-Beta (Not Launching With These)

- Chat-driven onboarding rebuild (V1 — replaces Day 6 form interim)
- Chat-driven UX rebuild for M1A/B/C/D flows (V1)
- M1D Strategy Coach (current 5-section prompt stays)
- Phase 2 search learning signal wiring
- Standalone STAR builder from profile (hidden in profile UI per `star_library_prd.md`)
- Multiple resumes per user
- **Multi-profile architecture (IC vs manager etc.)** — NEW May 20, V1 priority
- **Two-resume engine human-readable version** — NEW May 20, V1 priority (ATS/system version ships closed beta)
- **Skills section in M1 framework** — NEW May 20, V1.5
- **Button rename pass** ("Create Opportunity", "My Opportunities") — NEW May 20, post-beta CSS sweep
- Career Themes auto-extraction on resume profile
- Job Seeking Status field beyond basic active/passive/employed
- HM chat auto-detect STAR content
- Per-section "Swap story" in HM prep output
- SerpAPI as automatic failover
- Public V1: Stripe, marketing site, public signup
- Admin dashboard polished UI
- Mobile-responsive nav
- Always-On Plan tier (V1.5 — career-cultivation pricing — see workdoc card)
- Full rail unification — LinkedIn-style filter bar (V1.5 — UX polish, not debt)

---

## PRD Queue — Specs to Write

### Tier A — Closed Beta Critical (DONE as of May 13)
- ✅ `search_engine_prd.md`
- ✅ `chat_framework_content_source_map_prd.md`
- ✅ `m1c_hm_interview_prd.md`
- ✅ `star_library_prd.md`
- ✅ `resume_engine_prd.md`
- ✅ `m1d_output_typology.md`
- ✅ `sales_deck_supermetrics_nebo_prd.md`
- ✅ `yelp_partner_outreach_strategy_prd.md`
- ✅ `discovery_call_yelp_prd.md`
- ✅ `brand_design_system_prd.md` (written May 13-14 by Marcelo)

### Tier B — Write as you start building each feature

Don't write upfront. Write the PRD in the SAME session as the feature build, before code.

- [ ] `m1a_my_opportunities_prd.md` — write when starting outreach package build (Day 7)
- [ ] `m1b_recruiter_interview_prd.md` — only if M1B needs spec changes during build
- [ ] `competitive_positioning_prd.md` — write before tester onboarding messaging
- [ ] `ai_first_chat_driven_ux_prd.md` — write when chat rebuild starts (post-beta)
- [ ] `path_a_mobile_share_prd.md` — write at start of Day 8-9 Path A build
- [x] `onboarding_prd.md` — written Day 6 Session 4 (covers form-based interim + Session 5 rail integration plan + V1 chat-driven rebuild spec)

### Tier C — Post-Beta (write when scaling becomes relevant)

- [ ] `infrastructure_prd.md` — Render proxy + Supabase + Resend + LightningPDF setup spec
- [ ] `cost_optimization_prd.md` — model routing, caching, Haiku/Sonnet decisions
- [ ] `user_profile_identity_prd.md` — when multi-resume support arrives
- [ ] `pricing_prd.md` — before Public V1 launch (Stripe + 14-day trial + Always-On tier)
- [x] `seo_aeo_strategy_prd.md` — STUB created May 21; expands when positioning is locked and Framer marketing site build starts (V1 sprint)

### PRD Discipline

- Write the PRD in the same session as building the feature, NOT upfront
- Update existing PRDs in the same session as behavior changes
- Each PRD: 200-500 lines, decision-oriented
- Each PRD references `_CONSTITUTION.md` for principles
- Each PRD documents content sources (see `chat_framework_content_source_map_prd.md`)

---

## Open Risks

| Risk | Impact | Mitigation |
|---|---|---|
| **Scoring quality remains poor after rewrite** | Testers churn on irrelevant results | Day 10 measurement; iterate before public V1 |
| **Form-based onboarding feels low-effort to testers** | Closed beta first impression weakens | Acceptable interim; chat-driven rebuild is named V1 priority |
| **Path A email parsing brittle** | Mobile share flow fails | Day 9 buffer + extensive test cases |
| **Supabase free tier hits limits during beta** | Service interruption | 5-7 testers won't hit Free tier limits; upgrade if needed |
| **JSearch coverage of LinkedIn < 80%** | Marketing claim broken | Day 10 dogfood validation; add SerpAPI parallel if <50% |
| **Marcelo burnout in late sprint** | Quality drops late, bugs ship | Hard stop at midnight; buffer days 11-12 |
| **MARCELO contamination scope expands beyond 3 known sub-problems** (NEW May 20) | Day 7.5 balloons, slips Day 7-8 work | ✅ RESOLVED May 21: pattern struck three sessions in a row (A, D, E). Session F embedded structural discipline — pre-swap + post-swap file-wide grep verification — and closed the contamination class with grep evidence. **Lesson for future audits:** when scope is set by "what we already know is broken," it tends to miss what we haven't looked for. Any sweep brief from May 21+ must instruct build chat to verify completeness via file-wide grep BEFORE assuming the named enumeration is exhaustive. |
| **Site URL fix blocks email confirmation re-enable** (NEW May 20) | Email confirmation toggle stays off through closed beta | Decision needed Day 7 Session 2: pull Netlify deploy forward to Day 7 OR ship closed beta with confirmation disabled and re-enable post-launch |
| **V1 ships later than 1-2 weeks post-beta** (NEW May 20) | Truthful-future marketing claim ("RG learns from your clicks") stays aspirational longer than committed; tester trust gap widens | At your observed ~2.1x slippage multiplier, realistic V1 = 2-3 calendar weeks post-beta. If V1 slips past 3 weeks, soften marketing copy to truthful-now framing immediately. Track Phase 2 signal-wiring as the gating V1 task — surface progress weekly to Marcelo. |
| **Streaming render harder to implement than estimated** (NEW May 20) | Day 9-10 scope balloons or ships without streaming | Streaming pattern is a ~30-line change inside existing `runM0Search` for-loop. If actual implementation exceeds 1h, STOP and reassess — either there's an architecture issue worth surfacing, or the estimate was wrong. Streaming is non-negotiable for Option C ship quality. |
| **Option C cost concentration at V1 scale** (NEW May 20) | ~$0.50/user/mo bulk scoring requires pricing tier to absorb | Closed-beta cost is negligible (~$5/month total). V1 pricing decision must absorb $1.00/user/mo all-in. Flag if V1 pricing model lands below $5/mo where margins squeeze. |
| **Driver-chat compliance-validating-by-deferral** (NEW May 22) | Real product principles get traded for sprint-discipline framing — exactly the failure mode driver chat exists to prevent | ✅ DOCUMENTED May 22: Marcelo proposed search-rail rework citing "if search is bad UX, it kills conversions" — same principle that drove Option C scoring lock (which pulled work forward despite ~1-day launch slip). Driver chat initially called it "V1 scope creep" and recommended deferral. Inconsistent application of the same principle. Marcelo's pushback caught it. **Lesson:** when sprint discipline ("don't expand scope") contradicts a previously-locked product principle ("search drives conversion"), reflect honestly which one is the failure mode. The earlier lock IS the precedent; sprint discipline doesn't override product principles, it sequences them. Search-UX work moved into the sprint as Day 7 Sessions 3-5 (search-rail rework) with launch slip to June 4-8. |
| **Driver-chat brief-prescription-by-speculation** (NEW May 22) | Briefs prescribe specific code paths the driver chat hasn't read, sending build chats down wrong fixes | ✅ DOCUMENTED May 22: Session 2.7 brief named `profile.fullName` (doesn't exist), speculated input IDs (`profFullName` vs actual `settingsName`), and prescribed an avatar render-function rewrite (real bug was in the call site, not the render function). Build chat read code first, pushed back on three premises, did the right fixes. **Lesson:** when driver chat writes briefs without reading the underlying code, prescribed fixes drift into speculation. Going forward — briefs should describe symptoms + desired outcomes, not prescribe specific code paths or named fields. Let build chat read the code and propose the fix structure. Driver chat's job is scope discipline + product decisions, not code prescription. |
| **LinkedIn legal exposure for displaying scraped data** (NEW May 22) | LinkedIn sends cease-and-desist or pursues legal action against RG for displaying LinkedIn-sourced job postings via JSearch | **Risk is real but bounded.** RG is NOT the scraper (JSearch is). RG has not agreed to LinkedIn's User Agreement (no LinkedIn business account, no fake accounts). The hiQ Labs precedent (Ninth Circuit + 2022 settlement) targets direct scrapers who created fake accounts and agreed to the User Agreement — RG's downstream-customer posture is structurally different. LinkedIn has not pursued downstream consumers of job-aggregation APIs to the same degree (would litigate the entire industry — Indeed, ZipRecruiter, Glassdoor, JSearch all act as similar aggregators). **Mitigations in scope:** (1) Day 10 direct-employer URL preference logic — prefer Greenhouse/Lever/Workday/employer-careers URLs over LinkedIn URLs in Apply CTA; (2) marketing copy review pre-V1 — replace "we search LinkedIn" with "we search the web for jobs"; (3) legal consult before V1 launch; (4) hard rule: never scrape LinkedIn directly. **Closed beta posture unchanged** — invite-only, no public marketing, no apparent exposure. **V1 posture requires the mitigations above.** Not a legal opinion — this is structural reasoning. Real lawyer consult is on V1 pre-launch checklist. |
| **Build-chat-as-better-diagnostician pattern** (NEW May 22) | Driver-chat briefs prescribe wrong root causes; build chat catches them via code reading | ✅ POSITIVE PATTERN noted May 22 — Session 2.8 declined to ship speculative fix for Bug 1 because code reads correct, refuted 2 of 3 brief hypotheses via pre-flight grep, diagnosed the actual bug (routing race in `authOnLoginSuccess`) as part of Bug 2 investigation. Session 2.9 refuted the brief's "audit-scope-too-narrow" hypothesis with expanded grep evidence, then surfaced an unrelated P0 incidentally. Session 2.10 enumerated 17 call sites / 32 matches vs the brief's 3-line surface-area estimate. **This is the right division of labor:** driver chat handles scope + product decisions; build chat handles code reading + root-cause diagnosis. **Lesson — when in doubt about a fix scope, write the brief describing SYMPTOMS only and let the build chat propose root causes.** Tradeoff worth marking: shipping speculative defensive fixes adds permanent dead code; declining-to-ship is the right discipline when the retest can produce deciding evidence. |
| **Hardcoded literals audit discipline class** (NEW May 22) | Production credentials (API tokens, session cookies) hardcoded in client-side JS ship publicly to every visitor | ✅ DOCUMENTED May 22: Session 2.9 surfaced incidentally that index.html lines 8855-8857 contained Marcelo's real Apify production API token + real LinkedIn `li_at`/`JSESSIONID`/`bscookie` auth cookies, hardcoded into bundled JS that shipped to every visitor of `app.rolegarden.com`. Live exposure. Session 2.10 deleted the entire Apify path (17 sites, 32 matches) + added defensive `localStorage.removeItem` scrub for returning visitors. **Day 7.5 closed the prompt-string MARCELO contamination class. This is a DIFFERENT contamination class we never audited: hardcoded literal values in JS initialization code.** Lesson — future sessions adding any infrastructure credentials must explicitly grep for credential patterns (`apify_api_`, `sk-ant-`, `eyJ...` for JWT, `li_at=`, `JSESSIONID`) BEFORE shipping. Add to `_CONSTITUTION.md` Pre-Launch Checklist. The Day 7.5 audit pattern (file-wide grep before assuming enumeration complete) extends here — but with credential-pattern targets, not prompt-language targets. |
| **Optimistic success messages firing before actual success** (NEW May 26) | Client code shows success alert when API call returns, not when actual delivery/work succeeds — masks real failures | ✅ DOCUMENTED May 26: Mobile Forgot Password testing revealed the green "reset email sent" alert fires for empty email submit, before Supabase rejects the request. Real bug — client lacks pre-call validation AND fires the success message optimistically. Lesson: success messages must fire after the awaited promise resolves successfully, NOT after the call is made. Audit other places in app where this pattern may apply: any async operation where the client confirms success before the operation actually completes. Session 6 fixes this specific instance; broader audit deferred to Day 8 cleanup or case-by-case as surfaced. |
| **Build chat self-audits against Constitution rules** (NEW May 26, POSITIVE PATTERN) | Discipline rules logged in Constitution are being internalized — build chat catches its own violations during the work, not after | ✅ DOCUMENTED May 26: Session 3 build chat first-pass wrote two Constitution Rule #3 violations: (a) `const _origRender = rgRailIntentRender; rgRailIntentRender = function(){_origRender(); ...}` wrap pattern, (b) IIFE `_rgRailHookTitlesIntoIntent` that wrapped `rgAddChip`/`rgRemoveChip` with `_origAdd`/`_origRemove`. **Build chat caught both on its own review, refactored to patch the actual bottleneck (`rgRenderChips`) directly, deleted the IIFE wrapper, and verified zero violation matches in final file.** Pattern documented in session summary as a self-noted "Lesson for myself: when restructuring a large namespace, the temptation to 'add a small adapter layer on top' reintroduces the override pattern. Better discipline: identify the actual bottleneck and patch THAT directly." **This is the discipline compounding.** Constitution rules are landing in build-chat behavior without explicit prompting. Worth marking as a positive trajectory for the project. |
| **Browser-load verification discipline (NEW)** (NEW May 26 — Constitution addition pending) | Node.js `--check` confirms parse-correctness but doesn't catch runtime undefined references. Sessions can ship "syntax-clean" code that crashes on first browser load. | ✅ DOCUMENTED May 26: Session 3 ran post-deletion grep for DELETED symbols (returned 0 matches — correct), and Node.js syntax check passed. But the namespace rewrite KEPT calls to `rgRailIntentRenderChips` that the rewrite never re-introduced. Session 3 shipped with `rgRailIntentRenderChips is not defined` thrown on every Search render. Session 3.1 fixed in surgical follow-up. **Lesson:** browser-load verification is the missing audit class — open deployed file, navigate affected surfaces, watch console for zero errors for 30+ seconds BEFORE declaring shipped. Different from "are deleted symbols still referenced" (which Session 3 did correctly) — this is "are referenced symbols still defined" (the inverse direction). Add to `_CONSTITUTION.md` Pre-Launch Checklist as standing discipline for every session that touches >50 lines of JS. |
| **Undefined-reference audit operationalized as tooling** (NEW May 26, POSITIVE PATTERN) | Discipline patterns can be ad-hoc applied OR turned into reusable tooling. The latter compounds in value. | ✅ DOCUMENTED May 26: Session 3.1 build chat didn't just acknowledge the new browser-load + undefined-reference audit discipline — it OPERATIONALIZED it as Python tooling. Script extracts every `<script>` block, enumerates top-level function defs (covers `function X(`, `const X = (...) =>`, `async function X(`, `var X = function`, `window.X = function`, object-method patterns), enumerates calls, subtracts JS builtins + DOM API + common method names denylist. 473 defs / 598 calls / 103 unknowns surfaced before P0 fix; manual triage identified 1 real undefined reference (`rgRailIntentRenderChips`); remaining 102 unknowns were string-content false positives. Post-fix re-run verified the fix was complete + P1a's 5 new functions didn't introduce new undefined references (502 unknowns unchanged). **The audit pattern is now reusable tooling for every future session.** Worth marking as a compounding-discipline trajectory — discipline classes that become tools are worth more than discipline classes that stay verbal. Other audit classes worth converting to tooling: hardcoded literals audit (Session 2.10), Constitution Rule #3 pattern audit (already grep-able), dead-code candidates (already Python-implementable per Session 3.1's exploratory work). |
| **Multi-file proxy deploys: dependencies first** (NEW May 27) | Multi-file commits to GitHub via web UI deploy individually, in commit order. Code committed before its dependencies fails to deploy. | ✅ DOCUMENTED May 27: Session 3.2 had a deploy failure when proxy.js (which `require`s `@supabase/supabase-js`) committed BEFORE package.json (which lists the dependency). Render auto-deployed proxy.js → `npm install` ran with OLD package.json → `@supabase/supabase-js` not installed → `MODULE_NOT_FOUND` → deploy crashed. Recovery was fast (commit package.json next → fresh deploy → succeeds). **Lesson:** for multi-file changes that involve dependency additions, commit order matters: dependencies first, code that uses them second. OR single-commit via local git push (cleanest). Driver chat earlier sequenced this poorly in the deploy walk-through; will pre-emptively flag this in future briefs that involve dependency additions to the proxy. |

---

## Files Referenced

- `_CONSTITUTION.md` — project rules
- `matchtalent_workdoc.html` — full decision history
- `features/search_engine_prd.md` — Search Engine PRD (584 lines + Day 6 Session 6 additions)
- `features/m1d_output_typology.md` — M1D output router
- `features/m1c_hm_interview_prd.md` — HM prep + STAR Phase 1/Phase 2 flow (updated May 21)
- `features/m1b_recruiter_interview_prd.md` — Recruiter screen prep (NEW May 21 — 6 sections, no STAR)
- `features/chat_framework_content_source_map_prd.md`
- `features/onboarding_prd.md` — Onboarding form-based interim + Session 5 rail plan + V1 chat-driven rebuild
- `features/resume_engine_prd.md` — to be updated Day 7.5 with ATS-vs-human + multi-profile decisions
- `features/star_library_prd.md` — STAR library + bulk import + builder gating
- `features/search_rail_rework_brief.md` — Scope brief for Day 7 Sessions 3-5 (search-rail rework, NEW May 22)
- `features/seo_aeo_strategy_prd.md` — STUB (May 21). SEO + AEO strategy. Expands when V1 sprint starts and positioning is locked.
- `sessions/2026-05-13-day5-prd-extraction.md` — Day 5 session summary
- `sessions/2026-05-16-day6-session2-save-for-later.md` — Day 6 Session 2 summary
- `sessions/2026-05-16-day6-session3-dedup-pipeline.md` — Day 6 Session 3 summary
- `sessions/2026-05-16-day6-session4-onboarding-intent.md` — Day 6 Session 4 summary
- `sessions/2026-05-16-day6-session5-rail-intent.md` — Day 6 Session 5 summary
- `sessions/2026-05-16-day6-session6-closeout.md` — Day 6 closeout
- `sessions/2026-05-20-day7-session1-resend-infra.md` — Day 7 Session 1 (Resend + Cloudflare DNS)
- `sessions/2026-05-20-throwaway-deploy-friend-test.md` — friend-test session (deploy not shipped; feedback captured; MARCELO + accordion bugs surfaced)
- `sessions/2026-05-20-day7p5-sessionA-marcelo-cleanup.md` — Day 7.5 Session A (MARCELO cleanup at scoreJobs + runM1 + runAutoM1)
- `sessions/2026-05-21-day7p5-sessionC-framework-audit.md` — Day 7.5 Session C (TA + HM prep framework audit)
- `sessions/2026-05-21-day7p5-sessionD-prep-framework-fixes.md` — Day 7.5 Session D (6 framework fixes at TA + HM prep)
- `sessions/2026-05-21-day7p5-sessionE-m1d-marcelo-fix.md` — Day 7.5 Session E (M1D pitch swap + audit-completeness gap surfaced)
- `sessions/2026-05-21-day7p5-sessionF-marcelo-final-sweep.md` — Day 7.5 Session F (final 3 swaps + contamination class verifiably closed)
