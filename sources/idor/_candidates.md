# IDOR: candidate writeups (Find + Filter steps)

Filter criteria: reasonably recent, shows real root cause and impact, ideally links a disclosed report.
Covet sub-variant diversity: numeric ID, UUID/GUID guessable or leaked, encoded/hashed ID,
mass assignment / object-field IDOR, IDOR in API (REST/GraphQL), file/attachment access,
export/report, "change email/password" / account takeover, tenant/org isolation break,
blind IDOR (write-only), method/verb or content-type swap, nested/second-order.
Target: 15 to 25 kept. Ask before scraping more than ~25 pages in one run.

Status: candidate | keep | drop (reason) | scraped

| # | Title | Author | URL | Platform | Date | Report link | Status |
|---|-------|--------|-----|----------|------|-------------|--------|
| 1 | IDOR when editing email leads to Account Takeover on Atavist | bugra | https://hackerone.com/reports/950881 | hackerone | 2020-11-18 | same | scraped: full |
| 2 | Insecure Direct Object Reference (IDOR) | bate5a | https://hackerone.com/reports/2487889 | hackerone | 2024-05-23 | same | scraped: full (PoC media redacted) |
| 3 | IDOR Leads To Account Takeover Without interaction (MTN) | theranger | https://hackerone.com/reports/1272478 | hackerone | 2022-09-04 | same | scraped: full |
| 4 | IDOR to Account Takeover (US DoD) | nagli | https://hackerone.com/reports/969223 | hackerone | 2020-09-29 | same | scraped: partial (DoD redacted; details on naglinagli.github.io/DoD_IDOR, see supplements) |
| 5 | IDOR: Account Deletion via Session (Mozilla) | z3phyrus | https://hackerone.com/reports/3154983 | hackerone | 2025-06-03 | same | scraped: full |
| 6 | IDOR in API applications (able to see any ...) (Automattic) | bugra | https://hackerone.com/reports/1695454 | hackerone | 2022-11-01 | same | scraped: full |
| 7 | IDOR price manipulation (Acronis) | spookhorror | https://hackerone.com/reports/1403176 | hackerone | 2021-11-30 | same | scraped: full |
| 8 | IDOR allows access to report details via reference.json | sm41ldrag0n79_mbbank | https://hackerone.com/reports/3862641 | hackerone | 2026-10-01 | same | scraped: partial (only H1 summary; $15,000) |
| 9 | Disclose Hidden Comments on Media Section hub.vroid.com (pixiv) | giwadaoud | https://hackerone.com/reports/2541962 | hackerone | 2026-01-18 | same | scraped: full |
| 10 | Unauth IDOR to mass account takeover (US DoD .edu) | sp1d3rs | https://hackerone.com/reports/685338 | hackerone | 2024-07-19 | same | scraped: full |
| 11 | OTP IDOR: one code verifies the wrong phone number | scriptjacker | https://scriptjacker.medium.com/otp-idor-when-one-code-verifies-the-wrong-phone-number-df0d2b6475f8 | medium | 2026-09-13 | ? | scraped: full |
| 12 | Tackling IDOR on UUID based objects | Veshraj Ghimire | https://medium.com/pentesternepal/tackling-idor-on-uuid-based-objects-71e8cb2dc265 | medium | 2024-01-31 | ? | scraped: full |
| 13 | The IDOR That Got Me My First $3K: delete files across tenants | Sid_x95 | https://medium.com/@Sid_x95/the-idor-that-got-me-my-first-3k-and-let-me-delete-files-across-tenants-be5a65021832 | medium | 2025-11-09 | ? | scraped: full |
| 14 | IDOR and Broken Access Control: private data exposure via UUID | Ahmed Qaramany | https://c0nqr0r.medium.com/idor-and-broken-access-control-risking-private-data-exposure-dd808412ed13 | medium | 2024-10-13 | ? | scraped: full (author notes AI-assisted draft) |
| 15 | Exploiting UUIDs in Account Takeover | dimpchubb | https://medium.com/@dimpchubb/exploiting-uuids-in-account-takeover-a-penetration-testers-guide-to-bypassing-insecure-token-96de9cc520a3 | medium | 2025-02-25 | n/a | scraped: full (methodology, not a single finding) |
| 16 | How I Chained Recon and IDOR to Access 100s of Credit Cards | XCriminal | https://medium.com/@XCriminal/how-i-chained-recon-and-idor-to-access-100s-of-credit-cards-0ca50eb82a74 | medium | 2025-06-15 | ? | scraped: full |
| 17 | How a sneaker order led me to an IDOR ($50) | 0xnexus | https://medium.com/@0xnexus0xfault0x/how-a-sneaker-order-led-me-to-an-idor-vulnerability-and-50-d3553a9f9136 | medium | 2026-10-28 | ? | scraped: full |
| 18 | IDOR on HackerOne Embedded Submission Form | pinoywhitehat | https://medium.com/pinoywhitehat/idor-on-hackerone-embedded-submission-form-9e59c6f044b3 | medium | 2024-06-19 | https://hackerone.com/reports | scraped: full (user-supplied download, cites source URL) |
| 19 | IDOR Is Not Just id=123: smarter IDOR hunting | Muhammed Asfan | https://medium.com/@MuhammedAsfan/idor-is-not-just-id-123-a-simple-guide-to-smarter-idor-hunting-5f58de1109dd | medium | 2026-01-03 | n/a | scraped: full |
| 20 | IDOR & Bypass Techniques: practical methodology | N0aziXss | https://medium.com/@N0aziXss/idor-bypass-techniques-a-practical-methodology-for-security-testing-47376bbf1ac2 | medium | 2026-07-13 | n/a | scraped: full |
| 21 | The Bug Bounty Playbook: IDOR (252 reports analyzed) | Aacle | https://medium.com/@Aacle/the-bug-bounty-playbook-idor-72038e510337 | medium | 2026-07-20 | n/a | scraped: full (user-supplied download, cites source URL) |
| 22 | Insecure direct object references (IDOR) | PortSwigger | https://portswigger.net/web-security/access-control/idor | portswigger | n/a | n/a | scraped: full |
| 23 | GraphQL API vulnerabilities (IDOR via unsanitized arguments) | PortSwigger | https://portswigger.net/web-security/graphql | portswigger | n/a | n/a | scraped: full |
| 24 | GraphQL IDOR Vulnerabilities: what they are and how to fix | Escape | https://escape.tech/blog/idor-in-graphql/ | personal-blog | 2023-09-14 | n/a | scraped: full |
| 25 | Insecure Direct Object References (PayloadsAllTheThings) | swisskyrepo | https://swisskyrepo.github.io/PayloadsAllTheThings/Insecure%20Direct%20Object%20References/ | github | n/a | n/a | scraped: full |

## Index pages (for more candidates, not writeups themselves)
- reddelexc/hackerone-reports (IDOR topic): https://github.com/reddelexc/hackerone-reports
- securehyperloop IDOR index: https://securehyperloop.com/bug-bounty/idor
- OWASP IDOR prevention cheat sheet: https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html

## Follow-ups from scrape (2026-11-16)
- Paywalled, not saved (user may supply): #18 https://medium.com/pinoywhitehat/idor-on-hackerone-embedded-submission-form-9e59c6f044b3 ($2,500); #21 https://medium.com/@Aacle/the-bug-bounty-playbook-idor-72038e510337.
- #4 DoD: author external blog (naglinagli.github.io/DoD_IDOR) is GONE (404) and no live copy found, so #4 stays partial.
- #8 HackerOne $15k: only H1 summary is public (reporter body not exposed).
- #24 escape.tech names H1 reports #415081, #42587, #56511 (GraphQL IDOR examples).
- 25/25 scraped: #18, #21 added from user-supplied downloads (2026-11-16). Raw files in sources/idor/.
