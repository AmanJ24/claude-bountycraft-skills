# case-06: Blind SQLi with conditional responses (PRACTITIONER) - TOOLING-BLOCKED

- **Lab:** https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses
- **Skill path:** SKILL.md section 3 (boolean-blind); references/blind-oob-and-second-order.md; notes 03, 18, 22, 24.
- **Date:** 2026-11-19, agent-run live.

## Intended solve (per the skill)
The `TrackingId` cookie is injected into a query whose result shows a "Welcome back" message only when it returns rows. Confirm the oracle with `TrackingId=xxx' AND '1'='1` (Welcome back) vs `'1'='2` (not), then extract the administrator password one character at a time with `' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1) > 'm'` and binary search, then log in.

## Why it is tooling-blocked here (not a skill gap)
The injection point is the `TrackingId` **cookie**, and this harness cannot set it:
- The Claude-in-Chrome extension blocks `document.cookie` with a `SecurityError` on the lab page (read and write), so the payload cannot be placed in the cookie from in-page JS.
- `fetch()` cannot set the `Cookie` request header (a forbidden header), so the payload cannot be delivered that way either.

Attempting to read/set `TrackingId` returned `SecurityError: Failed to read the 'cookie' property from 'Document': Access is denied`. This is the same documented limitation that blocked the business-logic encryption-oracle lab. All PortSwigger blind SQLi labs (conditional responses/errors, time delays, OAST) use the cookie injection point, so the same block applies; the OAST variants additionally need Burp Collaborator (not available in-harness).

## Status
Technique PROVEN by sources, not green-bannered. The boolean-blind extraction method is fully documented (references/blind-oob-and-second-order.md) and sourced from real findings: note 03 (PortSwigger boolean), 18 (DoD User-Agent boolean-blind), 22 (Vaadata boolean/time), 24 (boumriga time-based). To green-banner, drive the cookie/Collaborator in Burp. The in-band channels (cases 01-05) are validated green.
