# Conference talk: SSRF through PDF generators and HTML renderers to cloud credentials

Source completeness: partial (slide text only; image-only slides, including the ERB-to-RCE payload and screenshots, were lost in parsing).

- **Vuln type / sub-variant:** Technique survey: HTML/script injection into server-side PDF generators giving SSRF, local file read, XXE and template injection, plus cloud metadata theft. Not a single finding.
- **Target type:** Features that generate PDFs or images from user-influenced HTML: invoices, reports, exports, profile or card renderers, with engines such as wkhtmltopdf, headless Chrome, WeasyPrint and Prince. Also general URL fetchers (avatar from URL).
- **Recon signal:** Any place user input ends up inside a generated PDF. Confirm HTML renders (a tiny write-in-script test or visible markup), check whether script runs, see whether a tag pointing at your server causes a hit, and read the User-Agent in that hit to identify the engine. Styling options (font names, colors) are also injection points. Template syntax probes showing evaluated math reveal template injection.
- **Root cause:** PDF conversion happens on the server in a browser or renderer, so injected markup executes or fetches in the server's network context, with access to metadata and internal hosts. Defenders also leave scripts enabled and do not restrict egress or metadata.
- **Exploitation steps:**
  1. Find a field reflected into a PDF; test HTML and script rendering.
  2. Trigger an outbound fetch to your host; capture the User-Agent to fingerprint the engine.
  3. For script-capable renderers: use an iframe, script redirect or similar to load internal URLs and cloud metadata; for HTML-only renderers pick engine-specific features.
  4. WeasyPrint case (found by reading its source): a link tag with rel set to attachment and a file or http URL embeds the fetched content as a hidden attachment in the PDF; unpack it to read it. This took roughly three months to work out and eventually produced cloud keys.
  5. Prince v10 case: an image tag leaks the UA; the engine had an XML external entity flaw (CVE-2018-19858), so an XXE entity pointing at internal URL reads data.
  6. Template-engine case: probe with expression syntax; if arithmetic evaluates, escalate toward RCE.
  7. For blind cases, exfiltrate with script, and use DNS rebinding tools (HTTPRebind, SSRFTest) for headless browsers.
- **Bypasses used:** (1) CSS/style breakout injecting HTML after a closing style tag when only styling input exists. (2) Open redirect chaining to reach internal URLs when iframes follow redirects or domains are allowlisted. (3) Custom attacker domain pointing at metadata IP to defeat IP filters. (4) Redirect hop from attacker PHP script. (5) DNS rebinding to defeat same-origin and IP checks in headless browsers. (6) Alternate protocols: file, gopher, ssh schemes in generic URL fetchers. (7) WeasyPrint attachment link technique. (8) XXE in Prince. (9) Server-side template injection (ERB) leading to RCE. (10) Exfil via script when output is not visible.
- **Impact:** Cloud metadata keys (AWS EC2 access keys obtained in WeasyPrint case), local file read, internal data, and in the template case RCE. Cites JIRA CVE-2017-9506 and Jenkins CVE-2018-1000600 as metadata-reaching SSRF precedents.
- **Bounty:** n/a (talk, no single bounty).
- **Lesson / pattern:** If a PDF generator exists, it is very likely an SSRF surface; identify the engine from the User-Agent and read its docs or source for engine-specific fetch features before giving up.
- **Source:**
  - Title: Owning the Cloud Through SSRF and PDF Generators (AppSec California 2020)
  - Author: Ben Sadeghipour and Chris Holt (with Cody Brocious)
  - URL: https://static.sched.com/hosted_files/appseccalifornia2020/11/Ben%20Sadeghipour%20-%20Owning%20the%20clout%20through%20SSRF%20and%20PDF%20generators%20-%20ORIGINAL.pdf
  - Date: 2020
  - Disclosed report: none
  - Raw file: `sources/ssrf/27-slides-owning-the-cloud-ssrf-pdf-generators.md`

Also in the talk: a vendor-side triage setup using internal canary hosts and logging to classify SSRF as open redirect, blind, content-based or full, and a defender recap (disable script, allowlist, restrict cloud instance access).

Tags: pdf-generator, headless-browser, cloud-metadata, dns-rebinding, html-injection, xxe
