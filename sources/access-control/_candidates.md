# Broken Access Control / Auth Bypass: candidate writeups (Find + Filter)

Find/Filter done 2026-11-17. Scope: authorization/access-control is the core (OWASP A01), plus
authentication-bypass mechanics. Object-level horizontal access (pure IDOR) is CROSS-REFERENCED to the
`idor` skill, not duplicated; flawed-state-machine auth bypass is cross-referenced to `business-logic`;
TOCTOU to `race-conditions`. Covet sub-variant diversity:
- Vertical privesc: unprotected admin functionality, security-by-obscurity URL, parameter/role-based
  (role=admin, profile role field), horizontal-to-vertical (password disclosure).
- Missing function-level access control (MFLAC): API authz missing though UI hides it (PUT/POST to API).
- Platform/URL/method/header bypass: 403/401 bypass (X-Original-URL, X-Rewrite-URL, path normalization
  %2e/%252e, method override, case/encoding), URL-based front-end enforcement, method-based.
- Multi-step process with no check on one step; referer-based access control.
- Auth bypass: JWT (alg=none / nonE case bypass, key confusion, kid), OAuth redirect_uri -> code/token
  theft -> ATO, login-flow/MFA/email-verification bypass, response-manipulation login bypass.
Target: 15 to 25 kept. Ask before scraping more than ~25 pages in one run. (24 selected.)

Status: candidate | keep | drop | scraped

| #  | Title | Author | URL | Platform | Sub-variant | Status |
|----|-------|--------|-----|----------|-------------|--------|
| 01 | Access control vulnerabilities and privilege escalation (topic + labs) | PortSwigger | https://portswigger.net/web-security/access-control | portswigger | taxonomy + labs (spine) | candidate |
| 02 | HTTP 401/403 Authorization Bypasses | HackTricks | https://hacktricks.wiki/en/pentesting-web/403-and-401-bypasses.html | hacktricks | 403/header/path/method bypass arsenal | candidate |
| 03 | How To Find Broken Access Control Vulnerabilities in the Wild | HackerOne | https://www.hackerone.com/blog/how-find-broken-access-control-vulnerabilities-wild | hackerone-blog | BAC methodology | candidate |
| 04 | OAuth to Account Takeover | HackTricks | https://hacktricks.wiki/en/pentesting-web/oauth-to-account-takeover.html | hacktricks | OAuth redirect_uri/mix-up/code reuse | candidate |
| 05 | OAuth 2.0 Vulnerabilities (cheat sheet) | 0xn3va | https://0xn3va.gitbook.io/cheat-sheets/web-application/oauth-2.0-vulnerabilities | gitbook | OAuth redirect_uri/host-poison/session | candidate |
| 06 | Authentication Bypass Security Guide (JWT/OAuth/SAML/session) | chs.us | https://chs.us/guides/authentication-bypass/ | other | JWT alg=none/key-confusion/kid taxonomy | candidate |
| 07 | 5 Unusual Authentication Bypass Techniques | Synack | https://www.synack.com/exploits-explained/exploits-explained-5-unusual-authentication-bypass-techniques/ | other | auth bypass techniques | candidate |
| 08 | Privilege Escalation: Read-Only to Admin (PUT to API despite hidden UI) | see report | https://hackerone.com/reports/277138 | hackerone | MFLAC: UI hides, API allows | candidate |
| 09 | Guest Privilege Escalation to admin group (Rocket.Chat) | gronke | https://hackerone.com/reports/501081 | hackerone | vertical privesc: improper ACL | candidate |
| 10 | Bypass front server restrictions via X-Rewrite-Url / X-Original-URL (Clario) | see report | https://hackerone.com/reports/737323 | hackerone | header bypass of /admin front-end block | candidate |
| 11 | Insufficient Access Control on Registration -> privesc to Administrator | see report | https://hackerone.com/reports/796379 | hackerone | registration privesc | candidate |
| 12 | Privilege escalation (GitLab) | see report | https://hackerone.com/reports/493324 | hackerone | vertical privesc | candidate |
| 13 | Privilege Escalation in Edit (Dust) | see report | https://hackerone.com/reports/3103755 | hackerone | privesc in edit/role | candidate |
| 14 | [H1-2006] Bypassing access control checks by ... (CTF writeup) | see report | https://hackerone.com/reports/895172 | hackerone | access-check bypass techniques | candidate |
| 15 | Broken Access Control (MTN) | see report | https://hackerone.com/reports/2319586 | hackerone | BAC real report | candidate |
| 16 | Login email verification bypass via /oauth/token (GitLab ROPC) | see report | https://hackerone.com/reports/2676025 | hackerone | auth bypass: ROPC skips verification | candidate |
| 17 | Authentication bypass in Auth0 (alg:nonE case-sensitive none) | Sentor Security | https://sentorsecurity.com/blog/vulnerability-disclosure-authentication-bypass-in-auth0/ | other | JWT none-alg case bypass | candidate |
| 18 | Stealing JWT Tokens via OAuth redirect_uri Manipulation | InfoSec Writeups | https://infosecwriteups.com/stealing-jwt-tokens-via-oauth-redirect-uri-manipulation-a-critical-vulnerability-abbd579b5443 | medium | OAuth redirect_uri -> token theft | candidate |
| 19 | OAuth 2.0 Redirect URI Bypass -> Account Takeover ($8,000) | t4nv1 | https://medium.com/@t4nv1/how-an-oauth-2-0-redirect-uri-bypass-led-to-account-takeover-and-an-8-000-bounty-ad67ba4b0db6 | medium | OAuth redirect_uri -> ATO | candidate |
| 20 | page.line.me Open Redirect -> OAuth code exposure -> token compromise ($1,000) | see report | https://hackerone.com/reports/3423013 | hackerone | OAuth code leak via open redirect | candidate |
| 21 | $800 Bounty: Privilege Escalation via API, Scheduler to Team Admin | InfoSec Writeups | https://infosecwriteups.com/800-bounty-privilege-escalation-via-api-from-scheduler-to-team-admin-810bb8401a0f | medium | API role privesc | candidate |
| 22 | From Nobody to Admin: a practical guide to privilege escalation | Muhammed Asfan | https://medium.com/@MuhammedAsfan/from-nobody-to-admin-a-short-practical-guide-to-privilege-escalation-b6749f3ec9a7 | medium | privesc methodology | candidate |
| 23 | Ultimate 401 and 403 bypass methods | Vidoc Security | https://blog.vidocsecurity.com/blog/401-and-403-bypass-how-to-do-it-right | other | 403 bypass methodology | candidate |
| 24 | Account Takeover Due to Improper Handling of JWT Tokens | huntr | https://huntr.com/bounties/4747a485-77c3-4bb5-aab0-21253ef303ca | other | JWT handling -> ATO | candidate |

Notes:
- Expect some HackerOne reports to hit the sign-in / limited-disclosure wall (business-logic lost 5;
  race-conditions lost 0). Research/reference (01-07, 17, 22, 23) is reliably public.
- Horizontal/object-level IDOR deliberately deferred to the `idor` skill; this skill covers the
  authorization-enforcement failures and auth bypass, and cross-references IDOR for object access.
- OAuth/JWT are represented (not exhausted); they could each become their own skill later. Flag in SKILL.
- Backlog if gaps surface: gobeecode/codebygk curated H1 access-control/auth lists; SAML-specific reports;
  the Shopify/lemlist/UPS admin-access reports (#1394982, #937921, #1539426).
