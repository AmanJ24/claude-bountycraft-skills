# Sources: Access Control / Auth Bypass skill

24 sources distilled into `extracted/access-control/`, synthesized into `SKILL.md` and the five `references/` files. Raw scrapes in `sources/access-control/`. Paraphrased, never copied; only citation lines overlap source text. Source #19 (t4nv1, member-only Medium) was supplied by the user as a legitimate download in `Manual_downloads/` and folded in citing the original URL.

Scope: this skill covers authorization-enforcement failures (vertical/horizontal privesc, missing function-level access control, 401/403 bypass) and authentication bypass (JWT, OAuth/OIDC, session, SSO). Pure object-level IDOR is cross-referenced to the `idor` skill; flawed-state-machine auth bypass to `business-logic`; TOCTOU/refresh-token races to `race-conditions`.

## References and taxonomy (no single finding)

| # | Title | Author | URL | Role |
|---|-------|--------|-----|------|
| 01 | Access control vulnerabilities and privilege escalation | PortSwigger | https://portswigger.net/web-security/access-control | taxonomy + labs (spine) |
| 02 | HTTP 401/403 Authorization Bypasses | HackTricks | https://hacktricks.wiki/en/pentesting-web/403-and-401-bypasses.html | 403/header/path/method arsenal |
| 03 | How To Find Broken Access Control Vulnerabilities in the Wild | HackerOne | https://www.hackerone.com/blog/how-find-broken-access-control-vulnerabilities-wild | BAC methodology |
| 04 | OAuth to Account Takeover | HackTricks | https://hacktricks.wiki/en/pentesting-web/oauth-to-account-takeover.html | OAuth/OIDC ATO taxonomy |
| 05 | OAuth 2.0 Vulnerabilities (cheat sheet) | 0xn3va | https://0xn3va.gitbook.io/cheat-sheets/web-application/oauth-2.0-vulnerabilities | OAuth client/server flaws |
| 06 | Authentication Bypass Security Guide | Carl Sampson (chs.us) | https://chs.us/guides/authentication-bypass/ | JWT/OAuth/SAML/session taxonomy |
| 07 | 5 Unusual Authentication Bypass Techniques | Synack | https://www.synack.com/exploits-explained/exploits-explained-5-unusual-authentication-bypass-techniques/ | 5 real auth-bypass patterns |
| 22 | From Nobody to Admin: a practical guide to privilege escalation | Muhammed Asfan | https://medium.com/@MuhammedAsfan/from-nobody-to-admin-a-short-practical-guide-to-privilege-escalation-b6749f3ec9a7 | privesc playbook |
| 23 | Ultimate 401 and 403 bypass methods | Klaudia Kloc (Vidoc Security) | https://blog.vidocsecurity.com/blog/401-and-403-bypass-how-to-do-it-right | 403 bypass methodology |

## Disclosed findings

| # | Title | Author / Program | URL | Disclosed report | Bounty |
|---|-------|------------------|-----|------------------|--------|
| 08 | Privilege Escalation: Read-Only to Admin (API PUT despite hidden UI) | foobar7 / Inflection | https://hackerone.com/reports/277138 | yes (partial scrape) | awarded, hidden |
| 09 | Guest Privilege Escalation to admin group (Rocket.Chat) | gronke | https://hackerone.com/reports/501081 | yes | see report |
| 10 | Bypass front-server restriction via X-Rewrite-Url / X-Original-URL (Clario) | reporter / Clario | https://hackerone.com/reports/737323 | yes | see report |
| 11 | Insufficient Access Control on Registration -> privesc to Administrator | reporter | https://hackerone.com/reports/796379 | yes | see report |
| 12 | Privilege escalation to admin when admin impersonates you (GitLab) | skavans / GitLab | https://hackerone.com/reports/493324 | 2020-08-26 | awarded, hidden; fixed 11.8.1 |
| 13 | Privilege Escalation in Edit (Dust) | reporter / Dust | https://hackerone.com/reports/3103755 | yes | see report |
| 14 | [H1-2006] Bypassing access control checks (CTF chain) | bcobain23 / h1-ctf | https://hackerone.com/reports/895172 | 2020-06-10 | CTF (none) |
| 15 | Broken Access Control (Horizontal Privilege Escalation) (MTN) | aliyueka / MTN Group | https://hackerone.com/reports/2319586 | 2025-01-31 | none (Medium 6.5) |
| 16 | Login email verification bypass via /oauth/token (GitLab ROPC) | cybxis / GitLab | https://hackerone.com/reports/2676025 | 2024-09-03 | none (closed Informative) |
| 17 | Authentication bypass in Auth0 (cross-tenant JWT issuance) | Laban Skollermark (Sentor) | https://sentorsecurity.com/blog/vulnerability-disclosure-authentication-bypass-in-auth0/ | vendor ack only | not stated |
| 18 | Stealing JWT Tokens via OAuth redirect_uri Manipulation | Shah kaif (@SKaif009) | https://infosecwriteups.com/stealing-jwt-tokens-via-oauth-redirect-uri-manipulation-a-critical-vulnerability-abbd579b5443 | none (disclosed+patched) | not stated |
| 19 | How an OAuth 2.0 Redirect URI Bypass Led to Account Takeover and an $8,000 Bounty | t4nv1 | https://medium.com/@t4nv1/how-an-oauth-2-0-redirect-uri-bypass-led-to-account-takeover-and-an-8-000-bounty-ad67ba4b0db6 | none (member-only; user-supplied) | $8,000 (High/Critical) |
| 20 | page.line.me Open Redirect -> OAuth code exposure -> token compromise | imnotr3al / LY Corp (LINE) | https://hackerone.com/reports/3423013 | 2026-06-02 (partial scrape) | $1,000 (High 7.1) |
| 21 | $800 Bounty: Privilege Escalation via API, Scheduler to Team Admin | Abhi Sharma (@a13h1) | https://infosecwriteups.com/800-bounty-privilege-escalation-via-api-from-scheduler-to-team-admin-810bb8401a0f | none | $700 + $100 retest |
| 24 | Account Takeover Due to Improper Handling of JWT Tokens (usememos/memos) | M Nadeem Qazi (@mnqazi) | https://huntr.com/bounties/4747a485-77c3-4bb5-aab0-21253ef303ca | CVE-2023-4696 | not stated (Critical 9.8) |

## Notes on provenance

- Member-only article (19): the user downloaded the full t4nv1 article legitimately to `Manual_downloads/` (project paywall rule: never bypass, the user retrieves it); folded in citing the original Medium URL, not the freedium mirror.
- Partial scrapes (08, 20): the public report rendered only the program summary/timeline; exploitation detail is distilled from that summary and flagged in-note as "source is partial".
- Disputed disposition (16): GitLab closed it Informative, arguing the root cause is the victim's password being shared; kept as a real-world lesson on alternate-grant verification gaps and disclosure disagreement.
- Cross-skill: object-level id swapping is owned by the `idor` skill; SSRF chains (note 14, and OAuth registration SSRF params) by the `ssrf` skill; TOCTOU/refresh-token races by `race-conditions`; forced-browsing-past-a-step by `business-logic`.
- OAuth and JWT are represented but not exhausted; each could become its own skill later (flagged in SKILL.md).
