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
- [x] 7. Iterate: applied case-03 gap to references/bypasses.md (short-form vs dword reliability; obfuscate blocked PATH segments, single vs double URL-encode).
- [x] sources.md complete (29 sources + extracted-note links)
- [ ] Committed

### Backlog
- [ ] IDOR
- [ ] XSS (stored/reflected/DOM)
- [ ] Auth bypass / broken access control
- [ ] Race conditions

## Future (not in pilot, do not build without explicit ask)
- Phase 2: assistive active testing (human fires requests)
- Phase 3: semi-autonomous, scope-file gated
