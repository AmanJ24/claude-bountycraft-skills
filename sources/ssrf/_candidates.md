# SSRF: candidate writeups (Find + Filter steps)

Filter criteria: reasonably recent, shows real root cause and impact, ideally links a disclosed report.
Target: 15 to 25 kept. Ask before scraping more than ~25 pages in one run.

Status: `candidate` | `keep` | `drop (reason)` | `scraped`

| # | Title | Author | URL | Platform | Date | Report link | Status |
|---|-------|--------|-----|----------|------|-------------|--------|
| 1 | SSRF via Analytics Reports | megaman_ | https://hackerone.com/reports/2262382 | hackerone | 2023-12-08 | same | scraped: full (PoC redacted) |
| 2 | Blind SSRF to internal services in matrix preview_link API | la_revoltage | https://hackerone.com/reports/1960765 | hackerone | 2023-04-26 | same | scraped: full |
| 3 | Half-blind SSRF in kube cloud-controller-manager upgraded to full | reeverzax | https://hackerone.com/reports/776017 | hackerone | 2020-10-30 | same | scraped: full |
| 4 | Full read SSRF via Lark Docs "import as docs" | sirleeroyjenkins | https://hackerone.com/reports/1409727 | hackerone | 2022-01-28 | same | scraped: partial (team summary only) |
| 5 | SSRF in pdf generator (DoD, HtUS) | codeprivate | https://hackerone.com/reports/1628209 | hackerone | 2022-09-14 | same | scraped: full |
| 6 | Blind SSRF on errors.hackerone.net via Sentry misconfig | chaosbolt | https://hackerone.com/reports/374737 | hackerone | 2018-07-04 | same | scraped: full |
| 7 | External SSRF + local file read via FFmpeg HLS video upload | ach | https://hackerone.com/reports/1062888 | hackerone | 2021-02-15 | same | scraped: full |
| 8 | SSRF through Connection test feature (Apache Airflow) | sayoojbkumar | https://hackerone.com/reports/2123113 | hackerone | 2023-09-12 | same | scraped: full, short (IBB program) |
| 9 | SSRF in webhook functionality | madara_ | https://hackerone.com/reports/2301565 | hackerone | 2024-01-30 | same | scraped: full |
| 10 | SMB SSRF in emblem editor exposes domain credentials | alexbirsan | https://hackerone.com/reports/288353 | hackerone | 2018-01-12 | same | scraped: partial (summaries only) |
| 11 | SSRF in Custom Integration Webhook discloses AWS metadata | mohammad_obaid | https://hackerone.com/reports/643278 | hackerone | 2020-03-23 | same | scraped: partial (summary + timeline only) |
| 12 | SSRF chained to internal host leading to second SSRF | bugdiscloseguys | https://hackerone.com/reports/826097 | hackerone | 2021-03-30 | same | scraped: full |
| 13 | SSRF in webhooks leads to AWS private keys disclosure (Omise) | honoki | https://hackerone.com/reports/508459 | hackerone | 2019-06-28 | same | scraped: full |
| 14 | SVG Server Side Request Forgery | floyd | https://hackerone.com/reports/223203 | hackerone | 2017-09-22 | same | scraped: full |
| 15 | SSRF mitigation bypass using DNS rebind | adrian_t | https://hackerone.com/reports/1369312 | hackerone | 2022-11-25 | same | scraped: full, short |
| 16 | Blind SSRF in Integrations via bug in Ruby's native resolver | edoverflow | https://hackerone.com/reports/287245 | hackerone | 2017-11-09 | same | scraped: full |
| 17 | SSRF via filter bypass due to lax checking on IPs | obitorasu | https://hackerone.com/reports/1702864 | hackerone | 2023-02-10 | same | scraped: full |
| 18 | SSRF leaking internal Google Cloud data through upload function | dphoeniixx | https://hackerone.com/reports/549882 | hackerone | 2019-12-13 | same | scraped: partial (2-line summary; real writeup on Medium, see follow-ups) |
| 19 | SSRF in headless Chrome with remote debugging | d1r3wolf | https://hackerone.com/reports/781295 | hackerone | 2020-02-04 | same | scraped: full |
| 20 | SSRF in Exchange leads to ROOT access in all instances (Shopify) | 0xacb | https://hackerone.com/reports/341876 | hackerone | 2018-05-23 | same | scraped: full |
| 21 | SSRF in api.slack.com via slash commands, bypassing protections | elber | https://hackerone.com/reports/381129 | hackerone | 2019-02-22 | same | scraped: full |
| 22 | SSRF in gitlab.com via project import | edoverflow | https://hackerone.com/reports/215105 | hackerone | 2017-11-09 | same | scraped: full, comments gap (CVE-2017-0904) |
| 23 | Five Bounties, One Bug: same SSRF via five techniques | Kayra Oksuz | https://medium.com/@oksuzkayra16/five-bounties-one-bug-exploiting-the-same-ssrf-via-five-unique-techniques-3f0adb7965d6 | medium | 2025-11-16 | ? | scraped: full |
| 24 | A Real SSRF Story from HackerOne (IPv6 + Redirects) | Red Darkin | https://medium.com/@red_darkin/a-real-ssrf-story-from-hackerone-featuring-ipv6-redirects-9aa5e2ad8c2e | medium | 2026-04-15 | ? | scraped: full (same bug as #9) |
| 25 | Blind SSRF on a blockchain explorer via Next.js image loader | Anshubind | https://medium.com/@anshubind89/%EF%B8%8F-when-an-image-loader-turned-into-a-backdoor-my-discovery-of-a-blind-ssrf-on-a-blockchain-94d33d6d5371 | medium | 2025-12-10 | ? | drop (thin, no disclosed report; raw file in _dropped/) |
| 26 | Portable Data exFiltration: XSS for PDFs | Gareth Heyes | https://portswigger.net/research/portable-data-exfiltration | portswigger | 2020-12-10 | n/a | scraped: full |
| 27 | Owning the cloud through SSRF and PDF generators (slides) | Ben Sadeghipour, Chris Holt | https://static.sched.com/hosted_files/appseccalifornia2020/11/Ben%20Sadeghipour%20-%20Owning%20the%20clout%20through%20SSRF%20and%20PDF%20generators%20-%20ORIGINAL.pdf | conference | 2020 | n/a | scraped: partial (image-only slides lost) |
| 28 | Bypassing SSRF Protection | Vickie Li | https://vickieli.medium.com/bypassing-ssrf-protection-e111ae70727b | medium | 2019-06-08 | n/a | scraped: full |
| 29 | Vimeo upload function SSRF (author writeup for #18) | dPhoeniixx | https://medium.com/@dPhoeniixx/vimeo-upload-function-ssrf-7466d8630437 | medium | ? | https://hackerone.com/reports/549882 | scraped: full (merged into note 18) |
| 30 | Ruby Resolv bug (author writeup for #16/#22, CVE-2017-0904) | edoverflow | https://edoverflow.com/2017/ruby-resolv-bug/ | personal-blog | 2017 | https://hackerone.com/reports/215105 | scraped: full (merged into notes 16, 22) |
| - | The Ultimate Guide to SSRF (lostsec) | lostsec | https://lostsec.medium.com/the-ultimate-guide-to-ssrf-hunting-bypassing-and-exploiting-in-modern-bug-bounty-a43fa34ba324 | medium | ? | n/a | drop (members-only paywall) |

## Index pages (for more candidates, not writeups themselves)
- HackerOne SSRF reports sorted by bounty: https://github.com/codebygk/hackerone_bug_bounty_reports/blob/master/results/based_on_vulnerability_type/server_side_request_forgery_ssrf.md
- HackerOne Hacktivity CWE-918: https://hackerone.com/hacktivity/cwe_discovery?id=CWE-918
- Pentester Land writeup index: not yet searched

## Follow-ups from scrape (2026-11-16)
- [done] #18 Vimeo: real writeup is https://medium.com/@dPhoeniixx/vimeo-upload-function-ssrf-7466d8630437 (not yet checked for paywall).
- [done] #22 GitLab / Ruby Resolv: author writeup at https://edoverflow.com/2017/ruby-resolv-bug/ fills the comments gap.
- #9 and #24 are the same bug (H1 report + author's Medium post); treat as one finding in extraction.
- [done] #25 dropped.
- Pentester Land index still not searched.
