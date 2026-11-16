# SSRF via HTML injection in HackerOne analytics report PDF export

Source completeness: partial (PoC redacted in the public report; mechanism comes from the team's fix summary and the researcher's impact comment)

- **Vuln type / sub-variant:** SSRF through HTML injection into a server-side HTML-to-PDF renderer, full read via rendered output (credentials from cloud metadata)
- **Target type:** PDF generator (analytics/report export), reflected content inside an error message
- **Recon signal:** Not stated in source. The researcher found that a template element name they controlled was echoed back inside a "missing template" error string, and that string was later fed into the PDF renderer.
- **Root cause:** When a requested template did not exist, the code built an error message containing the attacker-supplied template name and passed it unescaped into the HTML that gets converted to PDF. The fix simply removed the user value from the message.
- **Exploitation steps:**
  1. Submit an analytics report request with a template element whose name contains an iframe tag pointing at an internal URL (exact PoC redacted).
  2. The server renders the error text as HTML while producing the PDF, so the renderer loads the iframe from the server side.
  3. Point the iframe at the cloud metadata service (169.254.169.254) and read the credentials from the generated PDF.
  4. Stop at proof of credential access and report immediately (the team praised stopping at that point).
- **Bypasses used:** HTML injection into renderer (iframe tag). No filter evasion mentioned.
- **Impact:** Temporary AWS credentials (access key, secret) from instance metadata. Researcher argued follow-on risk of resource manipulation, data loss, account takeover and command execution; the team confirmed credential exposure and rated CVSS 10.0. No evidence of prior exploitation.
- **Bounty:** $25,000
- **Lesson / pattern:** Any user value reflected into content that a server-side renderer turns into PDF (even inside an error string) is an SSRF/metadata sink; test error and "not found" paths, not just happy-path fields.
- **Source:**
  - Title: Server Side Request Forgery (SSRF) via Analytics Reports
  - Author: megaman_
  - URL: https://hackerone.com/reports/2262382
  - Date: 2023-12-08 (reported 2023-11-23)
  - Disclosed report: https://hackerone.com/reports/2262382
  - Raw file: `sources/ssrf/01-h1-2262382-analytics-reports-ssrf.md`

Tags: pdf-generator, html-injection, cloud-metadata, aws, full-read, error-reflection
