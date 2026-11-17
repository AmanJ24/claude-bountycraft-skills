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

### Backlog
- [ ] XSS (stored/reflected/DOM)
- [ ] Auth bypass / broken access control
- [ ] Race conditions

## Future (not in pilot, do not build without explicit ask)
- Phase 2: assistive active testing (human fires requests)
- Phase 3: semi-autonomous, scope-file gated
