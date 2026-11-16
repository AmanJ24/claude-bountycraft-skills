# Full-read SSRF in Lark Docs "import as docs" feature

Source completeness: partial (report body empty in the public JSON; only the vendor's one-paragraph summary and title/metadata are available)

- **Vuln type / sub-variant:** Full-read SSRF via a document import feature (per title); technique details not stated in source
- **Target type:** Document import / "import as docs" function in a collaboration suite
- **Recon signal:** not stated in source
- **Root cause:** not stated in source. The vendor says the feature could be used to reach services on the internal network.
- **Exploitation steps:** not stated in source
- **Bypasses used:** not stated in source
- **Impact:** Access to services running on the internal network (vendor summary). Severity rated critical (9.6). The title claims the response content could be read back (full read).
- **Bounty:** $5,000
- **Lesson / pattern:** Import-from-URL or import-file features in document suites are worth testing as SSRF carriers, especially when the importer returns converted content to you.
- **Source:**
  - Title: Full read SSRF via Lark Docs `import as docs` feature
  - Author: sirleeroyjenkins
  - URL: https://hackerone.com/reports/1409727
  - Date: 2022-01-28 (reported 2021-11-24)
  - Disclosed report: https://hackerone.com/reports/1409727
  - Raw file: `sources/ssrf/04-h1-1409727-lark-docs-import-ssrf.md`

Tags: import-from-url, full-read, document-suite, internal-access
