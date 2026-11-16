# IP filter bypass with Unicode enclosed-alphanumeric and fullwidth address forms (Nextcloud)

Source completeness: partial (white-box code review plus a local test harness; exact end-to-end exploit on the live product and screenshots not available as text)

- **Vuln type / sub-variant:** SSRF filter bypass found by reading the open-source IP-check function; a host string using Unicode digit lookalikes and fullwidth dots gets past the check but is later normalized to the metadata IP.
- **Target type:** Server-side URL fetcher using the product's local-address guard (open-source cloud collaboration server).
- **Recon signal:** A recent related SSRF fix in the same code (including an added range for the Alibaba metadata IP) invited a second look at how complete the filter was. The reporter then reviewed the guard code and tried "advanced" encodings.
- **Root cause:** The guard validated the string with PHP IP validation functions and a local-range list, plus a special case for IPv6 with nested IPv4. A hostname that is not a valid IP literal under those functions skipped the checks, yet the HTTP stack later converted it to a real address. Unicode circled numerals and ideographic or fullwidth dots were not handled.
- **Exploitation steps:**
  1. Read the guard function and note it only rejects strings that validate as IP addresses.
  2. Build a local replica of the function that prints a pass message when no exception is raised, and feed it candidate payloads.
  3. Confirm that dotted AWS metadata address and the Alibaba address are rejected, public resolvers pass, and a circled-digit variant of 169.254.169.254 with ideographic full stops passes.
  4. Implied but not demonstrated in source: submitting such a host string to the real fetcher so that it normalizes to the metadata address.
- **Bypasses used:** Unicode enclosed alphanumerics with fullwidth/ideographic dot separators (encoding that normalizes later to 169.254.169.254). Dotted, decimal and IPv6-nested forms were already handled.
- **Impact:** Potential reading of cloud metadata on AWS-hosted instances. Real-world demonstration on a live instance not stated in source; proof was on the replica harness. Rated medium.
- **Bounty:** $250
- **Lesson / pattern:** When source is available, check where validation happens relative to normalization: any host string that "is not an IP" to the validator but resolves or normalizes to one later is a bypass.
- **Source:**
  - Title: SSRF via filter bypass due to lax checking on IPs
  - Author: obitorasu
  - URL: https://hackerone.com/reports/1702864
  - Date: 2023-02-10 (disclosed; submitted 2022-09-17)
  - Disclosed report: https://hackerone.com/reports/1702864
  - Raw file: `sources/ssrf/17-h1-1702864-ip-filter-bypass-nextcloud.md`

Tags: filter-bypass, unicode-encoding, cloud-metadata, white-box, ip-validation, open-source
