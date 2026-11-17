# XSS: candidate writeups (Find + Filter steps)

Standard: expert-level + real-world (see skill-quality-bar memory). Deliberately target the HARD
sub-variants, not trivial reflected XSS. Coverage goals:
- Reflected / stored / DOM basics (one or two canonical, for grounding)
- DOM XSS: sources/sinks, postMessage/web messages, hashchange, location-based
- CSP bypass (nonce reuse, JSONP, allowed-host gadget, dangling markup, strict-dynamic)
- Mutation XSS (mXSS) and sanitizer bypass (DOMPurify bypasses, innerHTML reparse)
- DOM clobbering
- Prototype pollution -> XSS (client-side gadget)
- Client-side template injection / AngularJS sandbox escape / framework (Vue, React dangerouslySetInnerHTML)
- Filter / WAF bypass, encoding, polyglots
- Blind / stored XSS with out-of-band exfiltration; XSS to account takeover (cookie/token theft, CSRF-token theft)
- SVG / markdown / file-upload XSS; PDF/email contexts
- Expert PortSwigger lab techniques (template literal, sandbox escape + CSP, strict CSP + dangling markup)

Target: 20 to 30 kept (broad class). Ask before scraping more than ~25 pages in one run.

Status: candidate | keep | drop (reason) | scraped

| # | Title | Author | URL | Platform | Date | Report link | Status |
|---|-------|--------|-----|----------|------|-------------|--------|
| 1 | Account takeover via cookie manipulation + XSS (Grammarly/Superhuman) | k4r4koyun | https://hackerone.com/reports/534450 | hackerone | 2019-12-03 | same | scraped: full (PoC redacted) |
| 2 | Cache poisoning -> stored XSS -> ATO (hav cookie, Abritel) | bombon | https://hackerone.com/reports/1760213 | hackerone | 2023-04-01 | same | scraped: full body (submitted 2022-11-02; no timeline) |
| 3 | Stored XSS on activity steals admin cookies (Shopify) | shazadsadiq | https://hackerone.com/reports/391390 | hackerone | 2018-08-14 | same | scraped: short body |
| 4 | Cross-account stored XSS to persistent administrator takeover (Basecamp) | matty69v | https://hackerone.com/reports/4175245 | hackerone | 2026-11-13 | same | scraped: full body (7 attachments not captured) |
| 5 | Blind stored XSS via contact form leaks session token + PII (Topcoder) | mase289 | https://hackerone.com/reports/878145 | hackerone | 2020-08-07 | same | scraped: full body (Topcoder, program lab45) |
| 6 | Blind stored XSS leads to admin account takeover | hemantsolo | https://hackerone.com/reports/1110243 | hackerone | 2021-03-11 | same | scraped: full body (DoD; host redacted) |
| 7 | DOM XSS via postMessage insecure handler (Lyst) | gamer7112 | https://hackerone.com/reports/1031644 | hackerone | 2021-02-09 | same | scraped: short body (rated LOW, not high) |
| 8 | DOM XSS via postMessage, origin not validated (Automattic/Jetpack) | renniepak | https://hackerone.com/reports/2371019 | hackerone | 2024-02-26 | same | scraped: full body |
| 9 | DOMPurify SVG mutation-XSS bypass (Internet Bug Bounty) | vovohelo | https://hackerone.com/reports/1024734 | hackerone | 2020-12-17 | same | scraped: full (IBB, no bounty; mXSS SVG) |
| 10 | Stored XSS + WAF bypass | jimgogogo | https://hackerone.com/reports/382625 | hackerone | 2018-12-07 | same | scraped: full (Semrush; WAF bypass payload kept) |
| 11 | Stored XSS via 64 vulnerable fields -> credential theft/ATO | z32 (un4gi) | https://hackerone.com/reports/847176 | hackerone | 2021-02-10 | same | scraped: full (DoD; host redacted) |
| 12 | Bypassing DOMPurify again with mutation XSS | Gareth Heyes | https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss | portswigger | 2020-10-07 | n/a | scraped: full (mXSS vectors kept) |
| 13 | Bypassing CSP via DOM clobbering | PortSwigger | https://portswigger.net/research/bypassing-csp-via-dom-clobbering | portswigger | 2023-06-05 | n/a | scraped: full (clobbering payloads kept) |
| 14 | XSS without HTML: Client-Side Template Injection with AngularJS | Gareth Heyes | https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs | portswigger | 2016-01-27 | n/a | scraped: full (sandbox escapes 1.0.1-1.6) |
| 15 | DOM based AngularJS sandbox escapes | PortSwigger | https://portswigger.net/research/dom-based-angularjs-sandbox-escapes | portswigger | 2017-05-11 | n/a | scraped: full |
| 16 | Widespread prototype pollution gadgets | Gareth Heyes | https://portswigger.net/research/widespread-prototype-pollution-gadgets | portswigger | 2022-06-22 | n/a | scraped: full (PP gadgets kept) |
| 17 | XSS cheat sheet (contexts, polyglots, sandbox escapes) | PortSwigger | https://portswigger.net/web-security/cross-site-scripting/cheat-sheet | portswigger | 2026-10-21 | n/a | scraped: full (contexts, sandbox escapes, polyglots) |
| 18 | Controlling the web message source (DOM XSS via postMessage) | PortSwigger | https://portswigger.net/web-security/dom-based/controlling-the-web-message-source | portswigger | ? | n/a | scraped: full |
| 19 | Client-side prototype pollution (Web Security Academy) | PortSwigger | https://portswigger.net/web-security/prototype-pollution/client-side | portswigger | ? | n/a | scraped: full |
| 20 | Mutation XSS via namespace confusion, DOMPurify <2.0.17 bypass | Michal Bentkowski | https://securitum.com/mutation-xss-via-namespace-confusion-dompurify-2-0-17-bypass.html | personal-blog | 2020-09-21 | n/a | scraped: full (namespace confusion mXSS) |
| 21 | From SVG and back: mutation XSS namespace confusion, DOMPurify <2.2.2 | vovohelo | https://vovohelo.medium.com/from-svg-and-back-yet-another-mutation-xss-via-namespace-confusion-for-dompurify-2-2-2-bypass-5d9ae8b1878f | medium | 2020-11-11 | n/a | scraped: full (not paywalled; 2 gists missing) |
| 22 | Exploring the DOMPurify library: bypasses and fixes | mizu.re | https://mizu.re/post/exploring-the-dompurify-library-bypasses-and-fixes | personal-blog | 2024-11-17 | n/a | scraped: full (part 1 of 2) |
| 23 | mXSS: The Vulnerability Hiding in Your Code | Sonar | https://www.sonarsource.com/blog/mxss-the-vulnerability-hiding-in-your-code/ | personal-blog | 2024-05-27 | n/a | scraped: full |
| 24 | Beyond XSS: Common CSP Bypasses | aszx87410 (Huli) | https://aszx87410.github.io/beyond-xss/en/ch2/csp-bypass/ | personal-blog | ? | n/a | scraped: full |
| 25 | Exploiting PostMessage Handlers to Achieve DOM XSS | Synack | https://www.synack.com/exploits-explained/exploiting-postmessage-handlers-dom-xss/ | personal-blog | 2026 | n/a | scraped: partial (code was images; prose kept) |
| 26 | CSP Bypasses: Advanced Exploitation Guide | Intigriti | https://www.intigriti.com/researchers/blog/hacking-tools/content-security-policy-csp-bypasses | personal-blog | 2026-10-15 | n/a | scraped: full-ish (some code was images; CSP strings kept) |
| 27 | CSP bypass on PortSwigger.net using Google script resources | joaxcar | https://joaxcar.com/blog/2024/02/19/csp-bypass-on-portswigger-net-using-google-script-resources/ | personal-blog | 2024-02-19 | hackerone (disclosed) | scraped: full (real finding; H1 2279346; $1000+$500) |
| 28 | Exploiting Client-Side Prototype Pollution in the wild | s1r1us | https://blog.s1r1us.ninja/research/PP | personal-blog | ? | n/a | scraped: full (real-world CSPP; bounties to $6000) |
| 29 | XSS Filter Evasion Cheat Sheet | OWASP | https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html | owasp | ? | n/a | scraped: full |
| 30 | XSS WAF Bypass: The Ultimate Deep Dive | SecurityTalent | https://github.com/SecurityTalent/write-up/blob/main/XSS%20WAF%20Bypass.md | github | 2026-06-08 | n/a | scraped: full (source ends mid-disclaimer) |

## Index pages (more candidates if needed)
- codebygk stored-XSS reports: https://github.com/codebygk/hackerone_bug_bounty_reports/blob/master/results/based_on_vulnerability_type/cross_site_scripting_xss_stored.md
- DOMPurify Attack Classes & Bypass History: https://github.com/cure53/DOMPurify/wiki/Attack-Classes-&-Bypass-History
- BlackFan client-side-prototype-pollution gadget corpus: https://github.com/BlackFan/client-side-prototype-pollution
- mXSS cheatsheet payloads: https://sonarsource.github.io/mxss-cheatsheet/examples/

## Expert PortSwigger XSS labs (for Step 6 validation)
- Reflected XSS with AngularJS sandbox escape and CSP (EXPERT): /web-security/cross-site-scripting/contexts/client-side-template-injection/lab-angular-sandbox-escape-and-csp
- Reflected XSS with AngularJS sandbox escape without strings (EXPERT): /web-security/cross-site-scripting/contexts/client-side-template-injection/lab-angular-sandbox-escape-without-strings
- Reflected XSS into a template literal ... Unicode-escaped (PRACTITIONER): /web-security/cross-site-scripting/contexts/lab-javascript-template-literal-...
- DOM XSS via client-side prototype pollution (PRACTITIONER) + third-party library variant
- Reflected XSS protected by CSP, with CSP bypass (EXPERT)

## Follow-ups from scrape (2026-11-16)
- 30/30 scraped, no paywalls. Bounty amounts hidden by HackerOne on most reports.
- #9 (HackerOne DOMPurify mXSS) and #21 (vovohelo Medium) are the same researcher (Daniel Santos); treat as complementary (report + author writeup) at extraction.
- #27 joaxcar is a real disclosed CSP-bypass finding (H1 2279346, $1000 + $500 form-action).
- #28 s1r1us: real-world client-side prototype pollution chains, bounties up to $6000.
- #22 mizu is part 1 of 2 (part 2: hunting-for-misconfigurations) if more depth needed later.
- Some code/diagrams were images and not captured (#25 Synack, #26 Intigriti, parts of #20/#21). Text payloads/CSP strings kept.
