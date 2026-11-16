# PDF injection and SSRF from inside generated PDFs (PortSwigger Research, Portable Data exFiltration)

Source completeness: full (research article; image-only parts omitted in the scrape, and SSRF is one section of a broader PDF-injection paper)

- **Vuln type / sub-variant:** Injection into PDF structure by libraries that fail to escape parentheses, enabling PDF JavaScript that sends blind POST requests (SSRF-like, plus DNS interaction) when the PDF is opened in Acrobat or PDFium
- **Target type:** Applications that build PDFs server-side with user input inside link annotations (npm libraries PDF-Lib and jsPDF were affected); also PDF upload features
- **Recon signal:** Any feature where user text or URLs end up in a generated PDF (tickets, invoices, receipts), especially hyperlink/annotation values. Test whether parentheses and backslashes survive unescaped, or can be produced via multi-byte characters.
- **Root cause:** The libraries escaped text streams but not annotation URI values, so an attacker who can include a closing parenthesis can add their own PDF dictionary entries, including actions and form fields.
- **Exploitation steps:**
  1. Identify: check escaping of parentheses/backslash and whether NUL, EOF markers or comments break the structure.
  2. Construct: break out of the URI value, attach a form widget with a parent dictionary, and run a JavaScript action (an alert proves it, a form submit to an external URL proves reach, useful when blind).
  3. Exploit: use the submitForm function with named text fields to issue a POST to an attacker or internal URL; raw newlines can be included, which could aid chaining with request smuggling.
  4. For exfiltration of PDF contents, submit the document itself to a remote server; in Acrobat a particular offline-dialog function triggers a DNS lookup without user approval (patched by Adobe).
- **Bypasses used:** HTML-style injection into PDF objects (parenthesis breakout), multi-byte character tricks to produce escape characters, hybrid payload covering both Acrobat and Chrome readers; on a separate PDF-upload case a WAF was dodged by loading a cached resource instead of making a fresh request.
- **Impact:** Blind SSRF (POST, response unreadable) from the PDF reader context, DNS-based data exfiltration, and theft of PDF contents viewed by other users. Note this runs in the viewer, not necessarily the generating server.
- **Bounty:** n/a
- **Lesson / pattern:** Treat PDF output as an injection context like HTML: test link annotations for unescaped parentheses, and remember reader-side requests can behave like blind SSRF.
- **Source:**
  - Title: Portable Data exFiltration: XSS for PDFs
  - Author: Gareth Heyes (PortSwigger Research)
  - URL: https://portswigger.net/research/portable-data-exfiltration
  - Date: 2020-12-10 (updated 2024-05-28)
  - Disclosed report: none
  - Raw file: `sources/ssrf/26-portswigger-portable-data-exfiltration.md`

Tags: pdf-generator, pdf-injection, blind, research, dns-exfil, filter-bypass
