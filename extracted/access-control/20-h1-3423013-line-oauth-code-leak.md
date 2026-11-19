# LINE page.line.me: open redirect in OAuth flow leaks authorization code, then access token (source is partial)

- **Vuln type / sub-variant:** open redirect inside an OAuth 2.0 authorization flow, leading to authorization-code exposure and access-token compromise (account takeover)
- **Target type:** page.line.me (LY Corporation / LINE) OAuth authorization flow
- **Recon signal:** redirect destinations on page.line.me were not restricted to trusted domains; an open redirect reachable from within the OAuth flow (source is partial, only the program summary is public)
- **Root cause:** redirect destinations not restricted to an allowlist of trusted domains; abused within the OAuth flow, the authorization response is sent to an attacker-controlled endpoint
- **Exploitation steps:** (program summary only; reporter's full PoC not in the public scrape)
  1. Abuse the open redirect on page.line.me so the OAuth authorization response is routed to an attacker-controlled endpoint.
  2. Capture the authorization code issued after the victim authenticates.
  3. Exchange that code for a valid access token.
  4. Use the token to act with the victim's privileges (unauthorized account access).
- **Bypasses used:** open redirect as the OAuth response-smuggling primitive (exact chain not public)
- **Impact:** unauthorized access to the victim's account, acting with the victim's privileges; rated High (7.1)
- **Bounty:** $1,000 (resolved and disclosed by LY Corporation)
- **Lesson / pattern:** an open redirect on or reachable from an OAuth authorization host is not low severity; it leaks the authorization code (or token) to the attacker and becomes account takeover. Restrict redirect destinations to trusted domains and never let the OAuth response follow an arbitrary redirect.
- **Source:**
  - Title: page.line.me Open Redirect Leading to OAuth Authorization Code Exposure and Access Token Compromise
  - Author: imnotr3al (reporter), LY Corporation (program)
  - URL: https://hackerone.com/reports/3423013
  - Date: 2026-06-02 (disclosed; reported 2025-11-12)
  - Disclosed report: https://hackerone.com/reports/3423013
  - Raw file: `sources/access-control/20-h1-3423013-line-oauth-code-leak.md`

Tags: oauth, open-redirect, authorization-code-leak, access-token, account-takeover, redirect-allowlist, source-partial
