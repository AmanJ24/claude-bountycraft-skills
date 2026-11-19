# SQL Injection evals (PortSwigger Web Security Academy)

Live-run against PortSwigger SQLi labs, user-authorized exception for PortSwigger labs only. Driven end to end via `navigate` + in-page `fetch`. In-band labs (auth bypass, UNION, examine-DB, filter-bypass) need no special tooling and reached the green "LAB Solved" banner. The blind labs inject via a cookie, which this harness cannot drive (see note).

## Results (agent-run live, 2026-11-19): 5/5 in-band PASS, 1 blind tooling-blocked

| Case | Lab | Level | Result | Skill branch validated |
|------|-----|-------|--------|------------------------|
| 01 | SQLi login bypass | APPRENTICE | **PASS** | auth bypass / logic subversion (`administrator'--`) |
| 02 | UNION, retrieve data from other tables | PRACTITIONER | **PASS** | UNION: column count + read `users` |
| 03 | UNION, multiple values in a single column | PRACTITIONER | **PASS** | UNION concat (`username\|\|'~'\|\|password`) |
| 04 | Query DB type and version (MySQL/Microsoft) | PRACTITIONER | **PASS** | examine DB / DBMS fingerprint (`@@version`) |
| 05 | SQLi with filter bypass via XML encoding | PRACTITIONER | **PASS** | WAF/filter bypass via hex-entity encoding |
| 06 | Blind SQLi with conditional responses | PRACTITIONER | **tooling-blocked** | boolean-blind (cookie injection point) |

Each in-band case was solved directly from the skill's documented methodology (SKILL.md sections 1-3, references/detection-and-contexts.md, in-band-union-and-error.md). No skill gaps surfaced.

### Case detail
- **01 login bypass:** username `administrator'--`, any password -> logged in as administrator. (SKILL.md section 1; note 01 logic subversion.)
- **02 UNION other tables:** `/filter?category=' UNION SELECT username, password FROM users--` -> read the admin credentials, logged in. (in-band-union-and-error.md.)
- **03 UNION single column:** `' UNION SELECT NULL, username||'~'||password FROM users--` (PostgreSQL concat) -> admin creds, logged in.
- **04 examine DB:** `' UNION SELECT @@version, NULL-- -` displayed `8.0.42-0ubuntu0.20.04.1` (MySQL). (dbms-specific-and-escalation.md fingerprint.)
- **05 XML-encoding filter bypass:** the plain `... UNION SELECT ...` in the `storeId` of the stock-check XML body was blocked (`403 "Attack detected"`); hex-entity-encoding every character (`&#x55;` for `U`, etc.) so the keyword filter could not see `UNION`/`SELECT`, while the XML parser decoded it before SQL, bypassed the filter and read the admin creds. Logged in. (waf-bypass-tooling-and-reporting.md; note 06/07 encoding bypass.)

## HARNESS LIMITATION (case 06, read this)
All PortSwigger **blind** SQLi labs (conditional responses, conditional errors, time delays, OAST) put the injection point in the `TrackingId` **cookie**. This harness cannot drive them:
- The Claude-in-Chrome extension blocks `document.cookie` (read and write) with a `SecurityError` on these pages, so the payload cannot be set in the cookie.
- `fetch()` cannot set the `Cookie` request header (a forbidden header), so the payload cannot be delivered that way either.

This is the same documented limitation that blocked the business-logic encryption-oracle lab. It is a harness constraint, not a skill defect: the boolean/time/OAST technique is fully documented in `references/blind-oob-and-second-order.md` and sourced from real findings (notes 03 PortSwigger, 18 DoD User-Agent boolean-blind, 22 Vaadata boolean/time, 24 boumriga time-based). The OAST labs additionally need Burp Collaborator (also not available in-harness; the Burp MCP was failed-to-connect this session). To green-banner the blind labs, drive them in Burp (edit the TrackingId cookie / Collaborator).

## Verdict
The skill is validated end to end on every channel this harness can drive: auth bypass, UNION (two variants), DBMS examination, and WAF/filter bypass, all green. The one non-green case is a cookie-injection harness limit with the technique sourced and documented, consistent with the honest precedent in the race-conditions and business-logic evals.
