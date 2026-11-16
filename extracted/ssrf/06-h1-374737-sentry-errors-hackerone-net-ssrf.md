# Blind SSRF through Sentry source-code scraping on a public error collector

Source completeness: full (curl body truncated in the scrape, but the mechanism is clear)

- **Vuln type / sub-variant:** Blind SSRF via a misconfigured third-party tool feature (Sentry "scrape source code")
- **Target type:** Self-hosted error/CSP report collector (Sentry) exposed to the internet
- **Recon signal:** The target's Content Security Policy contained a report endpoint with a sentry_key parameter, revealing Sentry. The hunter knew Sentry's UI error ingestion path is public and that the optional source-scraping setting fetches script URLs named in error events.
- **Root cause:** Sentry's source-code scraping option was enabled, so the server fetched whatever URL appeared in the filename field of a submitted event, with no restriction to external hosts (the researcher noted it could reach intranet addresses).
- **Exploitation steps:**
  1. Identify the Sentry key and project from the CSP report URI.
  2. Send a crafted error event to the store endpoint (instead of the CSP-report endpoint) with the stack frame filename set to an attacker-controlled URL.
  3. Watch the attacker server logs for the inbound GET from the Sentry host.
- **Bypasses used:** none (used the alternate public ingestion endpoint to reach the vulnerable code path)
- **Impact:** Blind outbound GET requests from the collector, possibly into internal networks. Researcher was unsure whether the host was firewalled; rated low (3.1).
- **Bounty:** $3,500
- **Lesson / pattern:** Spot third-party tooling from CSP and header hints, then recall its known SSRF-capable settings; public ingestion endpoints that take attacker-controlled URLs in event data deserve a callback test.
- **Source:**
  - Title: Blind SSRF on errors.hackerone.net due to Sentry misconfiguration
  - Author: chaosbolt
  - URL: https://hackerone.com/reports/374737
  - Date: 2018-07-04 (reported 2018-06-30)
  - Disclosed report: https://hackerone.com/reports/374737
  - Raw file: `sources/ssrf/06-h1-374737-sentry-errors-hackerone-net-ssrf.md`

Tags: blind, sentry, third-party-misconfig, error-ingestion, csp-recon, oob-callback
