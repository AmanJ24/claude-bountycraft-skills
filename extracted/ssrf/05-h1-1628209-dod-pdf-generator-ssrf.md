# SSRF through script injection in a PDF checklist generator (US DoD)

Source completeness: full (hostnames and credential values redacted)

- **Vuln type / sub-variant:** SSRF via JavaScript/HTML injection into a server-side PDF renderer, full read of cloud metadata in the output PDF
- **Target type:** PDF generator (dynamic checklist form printed from stored session data)
- **Recon signal:** The app offered a "view PDF" step that produced a PDF at a session-specific URL containing the data the user had typed in. A save API request carried a JSON object with the user-entered fields, so those fields flowed into a server-built document.
- **Root cause:** A field in the saved JSON (a name value) was embedded into the page that the server renders to PDF without escaping, so injected script ran in the renderer on the server.
- **Exploitation steps:**
  1. Walk through the questionnaire up to the action items step and open the PDF view.
  2. Intercept the save API call and send it to a repeater.
  3. Replace the name field in the global info object with a payload that closes the existing script block and writes an iframe pointing at the cloud metadata IAM credentials path.
  4. Confirm the API responds OK, then reload the generated PDF URL.
  5. The PDF shows the metadata response including the temporary key material.
- **Bypasses used:** HTML/script injection into renderer (breaking out of an existing script tag, then document.write of an iframe). No filter evasion mentioned.
- **Impact:** Exposure of AWS temporary credentials (access key, secret, token) for the instance role; rated critical (9.4).
- **Bounty:** $4,000
- **Lesson / pattern:** If a value you submit appears in a generated PDF, test for markup/script injection and point the renderer at 169.254.169.254; look at the JSON save calls behind print views.
- **Source:**
  - Title: SSRF in Functional Administrative Support Tool pdf generator
  - Author: codeprivate
  - URL: https://hackerone.com/reports/1628209
  - Date: 2022-09-14 (reported 2022-07-06)
  - Disclosed report: https://hackerone.com/reports/1628209
  - Raw file: `sources/ssrf/05-h1-1628209-dod-pdf-generator-ssrf.md`

Tags: pdf-generator, html-injection, cloud-metadata, aws, full-read, stored-input
