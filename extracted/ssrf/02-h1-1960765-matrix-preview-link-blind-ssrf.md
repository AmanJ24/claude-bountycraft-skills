# Partially blind SSRF in Matrix chat link-preview endpoint (Reddit)

Source completeness: partial (target hostnames and escalation details redacted in the public report)

- **Vuln type / sub-variant:** Partially blind SSRF via link-preview fetcher, with limited data returned (page title metadata)
- **Target type:** Link preview / URL unfurl API (Matrix media preview_url endpoint backing a chat product)
- **Recon signal:** The chat product was built on Matrix software, which ships a preview_url feature that fetches a user-supplied URL. Not stated in source how the hunter spotted it beyond that.
- **Root cause:** The preview endpoint did not restrict the requested URL to external hosts, so the homeserver would fetch internal addresses.
- **Exploitation steps:**
  1. Call the media preview_url endpoint with the url parameter set to an internal address.
  2. Read the og:title (and similar preview metadata) in the response to learn what the internal service is.
  3. Repeat across hosts and ports to enumerate internal services (response time varied, retrying was sometimes needed).
  4. Stop and ask permission before escalating further (the researcher declined to probe a possible RCE path without approval).
- **Bypasses used:** none
- **Impact:** Internal service enumeration and port scanning; only service names and IPs could be pulled out before the fix. Researcher suggested possible escalation but did not test it.
- **Bounty:** $6,000
- **Lesson / pattern:** Link-unfurl and preview endpoints are classic SSRF carriers; even when only a title leaks, that is enough for internal service discovery and a high-severity rating.
- **Source:**
  - Title: Blind SSRF to internal services in matrix preview_link API
  - Author: la_revoltage
  - URL: https://hackerone.com/reports/1960765
  - Date: 2023-04-26 (reported 2023-04-24)
  - Disclosed report: https://hackerone.com/reports/1960765
  - Raw file: `sources/ssrf/02-h1-1960765-matrix-preview-link-blind-ssrf.md`

Tags: link-preview, unfurl, blind, partial-read, internal-enum, chat
