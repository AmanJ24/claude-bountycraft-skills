# PortSwigger Academy access control topic: taxonomy and testable patterns

- **Vuln type / sub-variant:** methodology/taxonomy (vertical, horizontal, context-dependent access control failures)
- **Target type:** generic web apps; admin panels, role-gated functions, multi-step workflows, URL/method-based front-end rules
- **Recon signal:** admin links hidden in UI only, role or admin flags in cookies/hidden fields/query strings, platform-level URL rules, ID parameters in URLs, wizard-style flows, endpoints guarded only by Referer
- **Root cause:** enforcement is missing, obscured, client-controlled, or applied inconsistently (one layer checks URL/method, another resolves the route differently; one step of a flow is unguarded)
- **Exploitation steps:**
  1. Browse straight to guessed or leaked admin URLs (/admin, robots.txt entries, URLs embedded in role-conditional JavaScript, wordlist brute force).
  2. Look for role stored in user-controlled places (hidden field, cookie, query like ?admin=true or ?role=1) and flip it; also try adding a role field to profile-update requests.
  3. Where a front-end rule blocks a URL, resend with a URL-override header instead of the real path.
  4. Swap the HTTP method on a restricted endpoint (e.g. GET instead of POST) to dodge method-keyed rules.
  5. Change the id parameter to another user; if GUIDs, harvest them from messages/reviews; check redirects for leaked body data.
  6. Target an admin via horizontal access, then read or reset their credentials (horizontal -> vertical).
  7. In multi-step flows, replay the final confirmation request directly with the needed parameters.
  8. For Referer-gated sub-pages, forge the Referer to the parent admin URL.
- **Bypasses used:** `X-Original-URL: /admin/deleteUser` and `X-Rewrite-URL` with request line set to `/`; method swap; case variation (`/ADMIN/DELETEUSER`); Spring `useSuffixPatternMatch` (`/admin/deleteUser.anything`, default-on before Spring 5.3); trailing slash (`/admin/deleteUser/`); forged Referer; GUID leakage; geo restrictions beaten via proxy/VPN/client geolocation spoofing
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** access control breaks wherever two components disagree about what the request means or where a check is assumed rather than enforced; test every role x every function x every route variant.
- **Source:**
  - Title: Access control vulnerabilities and privilege escalation
  - Author: PortSwigger Web Security Academy
  - URL: https://portswigger.net/web-security/access-control
  - Date: unknown (scraped 2026-11-17)
  - Disclosed report: none
  - Raw file: `sources/access-control/01-portswigger-access-control-topic.md`

Tags: taxonomy, methodology, vertical-privesc, horizontal-privesc, horizontal-to-vertical, unprotected-admin, security-by-obscurity, parameter-role, url-based, method-based, header-bypass, path-normalization, case-encoding, multi-step, referer-based, defense
