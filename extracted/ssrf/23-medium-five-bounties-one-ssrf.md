# One blind SSRF, five successive filter bypasses (WooCommerce shop URL)

- **Vuln type / sub-variant:** Blind SSRF with timing-based internal port scanning, repeatedly re-exploited after each incremental patch.
- **Target type:** E-commerce channel integration (WooCommerce connect) where a GraphQL mutation takes a shop URL and the server probes the store's REST API path on it. Target name redacted.
- **Recon signal:** A "connect your store" field that the backend validates by requesting the supplied address. No response body is returned, but connect success or failure and latency differ, which exposes behavior.
- **Root cause:** The user-supplied shop address went straight into the server-side HTTP client with no allowlist. Each fix was a narrow string or range check on the symptom (specific IP text, then loopback range) rather than a design that validates the final resolved destination after redirects. The author's guesses about each fix are inferred, not confirmed by the vendor.
- **Exploitation steps:**
  1. Submit loopback with an open port versus a closed port; compare response time (fast when open, slow or timeout when closed) to scan internal ports.
  2. After the plain loopback string was blocked, use another address inside the loopback block (a 127.x address other than .1).
  3. After the whole loopback range was blocked, write loopback in octal dotted form with padded octets.
  4. After normalization of IPs was added, supply a URL on a public redirect service that responds with a redirect to localhost; the server follows it and the same timing scan works.
  5. After loopback was blocked on the redirect destination, use the same redirect service to point at the cloud metadata IP and private RFC1918 ranges; reachability was inferred from behavior since no body is shown.
- **Bypasses used:** (1) None initially, plain loopback with port scan. (2) Alternate address inside loopback CIDR (blocklist matched one literal only). (3) IP notation obfuscation, octal form defeating string comparison. (4) Redirect via open-redirect or redirect-as-a-service hop to localhost (destination not re-checked). (5) Redirect to cloud metadata 169.254.169.254 and private ranges, because only loopback was blocked after redirects.
- **Impact:** Internal port scanning of localhost and indications of reachability to metadata and private networks; no data read, blind only. Five separate bounty payouts claimed, amounts not given.
- **Bounty:** not stated in source (five bounties claimed, proof is a screenshot only).
- **Lesson / pattern:** After a patch, rerun the whole bypass ladder (other loopback addresses, notation variants, redirect hop, other private ranges); and for blind SSRF use latency and success/failure differences as the signal.
- **Source:**
  - Title: Five Bounties, One Bug: Exploiting the Same SSRF via Five Unique Techniques
  - Author: Kayra Oksuz
  - URL: https://medium.com/@oksuzkayra16/five-bounties-one-bug-exploiting-the-same-ssrf-via-five-unique-techniques-3f0adb7965d6
  - Date: 2025-11-16
  - Disclosed report: none
  - Raw file: `sources/ssrf/23-medium-five-bounties-one-ssrf.md`

Tags: blind, filter-bypass, port-scan, redirect, ip-obfuscation, cloud-metadata
