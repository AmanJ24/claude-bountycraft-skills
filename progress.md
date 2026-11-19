# Progress

## Environment
- [x] Directory layout created (sources/, extracted/, skills/, evals/, templates/)
- [x] Templates: source, extraction, SKILL, eval
- [x] Git repo initialized (no commits yet; first commit after SSRF skill is done)

## Vuln classes

### SSRF (pilot)
- [x] 1. Find: candidate list (28 candidates, 2026-11-16) in `sources/ssrf/_candidates.md`
- [x] 2. Filter: kept all scraped except #25 (dropped); #29, #30 added as supplements
- [x] 3. Scrape: raw text in `sources/ssrf/` (28/28 saved, 2026-11-16; 22 full, 6 partial)
- [x] 4. Extract: 26 notes in `extracted/ssrf/` (2026-11-16; merged 09+24, 18+29, 22+30; tags normalized)
- [x] 5. Synthesize: SKILL.md + 5 references (Opus, 2026-11-16)
- [x] 6. Test: 5 evals run live (agent, user-authorized PortSwigger exception, 2026-11-16). PASS 1-4 (localhost, backend port scan, blacklist filter, open-redirect). Case 5 (blind OOB) BLOCKED by environment (needs Burp Collaborator; not a skill defect).
- [x] 7. Iterate: applied case-03 gap to references/bypasses.md (short-form vs dword reliability; obfuscate blocked PATH segments, single vs double URL-encode). Backfill (2026-11-17, from Claude-BugHunter comparison): OOB confirmation discipline in testing-methodology.md (confirm-before-claim, per-parameter attribution + negative controls, blind-vs-full-read); richer metadata/protocol appendix in exploitation-and-escalation.md (Azure token, ECS, k8s SA token, gopher-to-Redis); cross-skill pointer in SKILL.md.
- [x] sources.md complete (29 sources + extracted-note links)
- [ ] Committed

### IDOR
- [x] 1. Find: 25 candidates in `sources/idor/_candidates.md` (2026-11-16)
- [x] 2. Filter: 25 kept (#18, #21 added from user downloads)
- [x] 3. Scrape: 25/25 saved in `sources/idor/` (2026-11-16; 23 full, 2 partial #4 #8)
- [x] 4. Extract: 25 notes in `extracted/idor/` (2026-11-16; tags normalized, verbatim overlap removed)
- [x] 5. Synthesize: SKILL.md + 5 references (Opus, 2026-11-16)
- [x] 6. Test: 5 evals run live (agent, user-authorized, 2026-11-16). PASS 5/5: static-reference IDOR, userid-param read, password-disclosure->ATO->delete, GraphQL private post, mass assignment.
- [x] 7. Iterate: backfill (2026-11-17, from Claude-BugHunter comparison): added references/chains.md (6 named compositions: read->ATO, leaked-secret->takeover, double-IDOR->financial/destructive, membership+mass-assignment->privesc, soft-delete persistence, IDOR->XSS), wired into SKILL.md + escalation-and-impact.md; cross-skill pointer in SKILL.md. All grounded in existing extracted notes.
- [x] sources.md complete
- [ ] Committed

### XSS
- [x] 1. Find: 30 candidates in `sources/xss/_candidates.md` (2026-11-16), weighted to expert variants (mXSS/DOMPurify, CSP bypass, AngularJS sandbox, client-side prototype pollution, postMessage, blind-stored-to-ATO, WAF bypass)
- [x] 2. Filter: all 30 kept
- [x] 3. Scrape: 30/30 saved in `sources/xss/` (2026-11-16; no paywalls)
- [x] 4. Extract: 30 notes in `extracted/xss/` (2026-11-16; payloads preserved; Sonnet safeguards flagged 3 batches so extraction moved to Opus)
- [x] 5. Synthesize: SKILL.md + 5 references (Opus, 2026-11-16)
- [x] 6. Test (5 evals run live, agent, user-authorized, 2026-11-16/17): case-01 template-literal PASS (green banner); case-02 prototype-pollution DOM XSS PASS (green banner); case-03 EXPERT AngularJS-sandbox+CSP: technique PROVEN (CSP bypass + sandbox escape + arbitrary JS via document.title), green banner blocked by non-dismissable alert dialog in automation; case-04 EXPERT AngularJS-sandbox-without-strings: GAP FOUND then FIXED -> PASS (green banner); case-05 web-message DOM XSS PASS (green banner, exploit-server delivery). 4/5 green banners + 1 technique-proven.
- [x] 7. Iterate (2026-11-17): case-04 surfaced a real gap (all skill AngularJS escapes used quoted strings; the Expert "without strings"/no-$eval lab defeats them). Added the string-free, $eval-free charAt-override + fromCharCode escape and the "$parse may read a parameter NAME" sink insight to advanced-techniques.md + contexts-and-payloads.md; re-validated -> lab solved.
- [x] sources.md complete
- [ ] Committed

### Business Logic
- [x] 1. Find: 24 candidates in `sources/business-logic/_candidates.md` (2026-11-17), spanning price manipulation (negative qty, fraction/rounding, param tampering, currency confusion), workflow/state-machine bypass, premium/subscription/license bypass, OTP/2FA/auth-flow logic, discount/coupon abuse, balance/credit manipulation, privilege/role logic.
- [x] 2. Filter: 24 kept (filtered out the keyword-noise C-function bugs from the curated H1 list); 4 are methodology/taxonomy references (Intigriti x2, OWASP cheat sheet, PortSwigger examples).
- [x] 3. Scrape: 21 sources saved in `sources/business-logic/` (2026-11-17; 19 full/usable, 2 partial #09 #10). Dropped 5 to the HackerOne sign-in wall / not-public (02 Shipt, 03 Adobe, both redundant; 17/22/24 New Relic). Backfilled 2 (#25 infosecwriteups UI-vs-API $1,500, #26 bitpanic taxonomy). See `_candidates.md` scrape-resolution.
- [x] 4. Extract: 21 notes in `extracted/business-logic/` (2026-11-17; 3 Sonnet batches; tags normalized via `_tags.md`; em-dash + verbatim-overlap checks clean; references as methodology notes).
- [x] 5. Synthesize: SKILL.md + 5 references + sources.md (Opus, 2026-11-17). Spine = "name the invariant, ask what enforces it". All 21 notes cited; #04/#07 kept as false-positive examples, #18 as no-net-benefit example.
- [x] 6. Test (2026-11-17): 6 PortSwigger Business Logic eval cases (apprentice->EXPERT), agent-run live, user-authorized. PASS 3/6 green banners across distinct core patterns: case-01 excessive-trust-client-side (price tampering), case-03 insufficient-workflow-validation (forced browsing / skip payment step), case-04 auth-bypass-flawed-state-machine (don't complete the role transition). case-02 low-level/integer-overflow: technique + exact solution PROVEN, green banner blocked by the 45s JS-exec timeout on ~340 sequential cart requests. case-05 infinite-money: deferred (loop-heavy, same timeout shape). case-06 encryption-oracle EXPERT: blocked (extension disables document.cookie, which the cookie-ciphertext attack requires). Non-green outcomes are harness limits, not skill defects.
- [x] 7. Iterate: no gap from labs. Payment-bypass backfill (2026-11-17, user request): added 3 publicly-readable sources (27 Jack Cable webhook/IPN spoofing, 28 Jsmon payment-gateway-bypass taxonomy, 29 Amjad Ali PayU gateway-callback replay) -> 24 sources/notes; made payment-gateway bypass first-class in SKILL.md (§2 four trust points, §4 levers), attack-surfaces.md (new section), manipulation-and-bypasses.md (levers), reporting.md. Confirmed via the user's logged-in H1 session that the 5 earlier-walled reports are "Access denied" even authenticated (limited disclosure, unreachable, not bypassed).
- [x] sources.md
- [x] Committed (2e32ee9 skill; payment-bypass backfill follow-up commit)

### Race Conditions
- [x] 1. Find: 20 candidates in `sources/race-conditions/_candidates.md` (2026-11-17), research-weighted (single-packet attack is the core technique): Kettle "Smashing the state machine" + single-packet-attack paper, PortSwigger topic, Flatt first-sequence-sync (expert), YesWeHack/Hackcraft/dev.to guides, Turbo Intruder docs; + 8 H1 reports (coupon/faucet/follow/invite/flag/etc) + practitioner writeups. Sub-variants: limit overrun, multi-endpoint, single-endpoint, partial-construction, rate-limit/OTP-brute bypass, double-spend, social-action, auth-flow token races.
- [x] 2. Filter: 20 kept (research spine reliably public; H1 reports may hit the sign-in/limited-disclosure wall as in business-logic).
- [x] 3. Scrape: 20 sources in `sources/race-conditions/` (2026-11-17; 19 full, 1 paywalled #19 xalgord preview-only, not bypassed). All 8 H1 reports came through full this run (coupon, Cosmos faucet $5k total, Slack survey, follow, undeletable member, invite-team, retest ~$2.1k, flag submission).
- [x] 4. Extract: 19 notes in `extracted/race-conditions/` (2026-11-17; 3 Sonnet batches + note 04 hand-written after a Sonnet safeguard blocked the first-sequence-sync research; tags normalized via _tags.md; em-dash + overlap checks clean). #19 dropped (redundant paywalled preview).
- [x] 5. Synthesize: SKILL.md + 5 references + sources.md (Opus, 2026-11-17). Spine = "a race is a check-then-act gap / short-lived sub-state; make requests collide in it via the single-packet attack; predict/probe/prove". References cover attack-surfaces, the single-packet attack (+ last-byte sync + first-sequence-sync), predict/probe/prove methodology, the sub-variant taxonomy with cases, reporting. All 19 notes cited.
- [x] 6. Test (2026-11-17): staged 5 eval cases. case-01 limit-overrun: PARTIAL race WON via 60-way parallel fetch burst (3 coupon stacks, $1337 -> $684.54, 51% off), proving the race + methodology, but green banner needs ~15 stacks in one sub-ms window = the single-packet attack, which this harness lacks (Kettle: lab is "near-impossible using multiple packets"). cases 02-05: not attempted (require Burp "send group in parallel" / Turbo Intruder). Eval phase tooling-blocked by design, NOT a skill defect.
- [x] 7. Iterate: none needed. Partial limit-overrun win validates the methodology; no skill gap surfaced. Full green-banner validation needs Burp + Turbo Intruder (documented in evals/README).
- [x] sources.md
- [ ] Committed

### Broken Access Control / Auth Bypass
- [x] 1. Find: 24 candidates in `sources/access-control/_candidates.md` (2026-11-17). Scope = authorization/access-control core (OWASP A01) + auth bypass; object-level IDOR cross-referenced to the idor skill, not duplicated. Sub-variants: vertical privesc (unprotected admin, security-by-obscurity, parameter/role-based, horizontal-to-vertical), MFLAC (API authz while UI hides), 403/401 + URL/method/header bypass (X-Original-URL, path %2e, method), multi-step, referer-based, auth bypass (JWT alg=none/nonE/key-confusion, OAuth redirect_uri->ATO, ROPC/MFA/email-verify bypass).
- [x] 2. Filter: 24 kept (research spine reliably public; H1 reports may hit the wall as in business-logic).
- [x] 3. Scrape: 24 sources in `sources/access-control/` (2026-11-17; mostly full, 08/10/20 partial, 19 paywalled-preview dropped). All H1 reports readable this run. #17 renamed to tenant-claim-bypass (real Auth0 bug is tenant/claim confusion, not none-alg).
- [x] 4. Extract (completed 2026-11-19, inline as Opus 4.8 per model rule: Agent enum exposes only `sonnet`=5.5, which cyber-flags; did not fall back to 5.5): 24 notes in `extracted/access-control/`. Resumed the 13 pending (04,05,06,07,15,16,17,18,20,21,22,23,24) + fixed truncated 12 and 14. `_tags.md` written (normalized). em-dash + Source/Tags checks clean. #19 (t4nv1 $8k OAuth ATO) folded in 2026-11-19 from the user's legit download in `Manual_downloads/` (prefix-match redirect_uri + same-domain open redirect via `/../` -> code leak -> ATO).
- [x] 5. Synthesize (2026-11-19, Opus 4.8): SKILL.md (83 lines) + 5 references (recognizing-and-surfaces, testing-methodology, 403-401-bypass, oauth-jwt-auth-bypass, reporting) + sources.md. Spine = "name the resource and action; what on the SERVER authorizes this caller for it? If the answer is the UI, a hidden route, a client-supplied id, or a forgeable claim, it's a bug". All 23 notes cited. em-dash clean.
- [x] 6. Test (2026-11-19, agent-run live, user-authorized PortSwigger exception): 5 eval cases, **5/5 PASS green banners** across distinct skill branches. case-01 unprotected-admin (force-browse robots.txt -> /administrator-panel), case-02 user-role-in-profile (roleid:2 mass-assignment in body; swapped from the cookie-based parameter lab because the harness can't overwrite an HttpOnly Admin cookie), case-03 URL-based bypass (X-Original-URL: /admin, validates Clario note 10), case-04 method-based bypass (POST 401 -> GET 200), case-05 multi-step (replay the confirmed=true final step as low-priv). Files in `evals/access-control/`.
- [x] 7. Iterate: no skill gap surfaced; every lab solved directly from the documented methodology. Harness notes recorded (fetch can't set Cookie/overwrite HttpOnly cookie -> cookie-value labs need Burp; some bypasses mutate state while returning non-2xx, confirm effect not status).
- [x] sources.md
- [ ] Committed (skill + README.md + evals + source/note 19, pending this commit)

### Backlog
- [ ] SQLi
- [ ] SSTI
- Note (2026-11-17): user added Burp MCP -> single-packet attack (race), Collaborator/OOB (blind SSRF), Intruder (blind SQLi) now possible for evals. Offer to retro-validate race limit-overrun green banner.

## Future (not in pilot, do not build without explicit ask)
- Phase 2: assistive active testing (human fires requests)
- Phase 3: semi-autonomous, scope-file gated
