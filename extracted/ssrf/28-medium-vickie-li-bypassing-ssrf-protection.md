# Guide: bypassing blocklist and allowlist SSRF protections

- **Vuln type / sub-variant:** Bypass technique catalogue for SSRF defenses (blocklist and allowlist). Not a single finding.
- **Target type:** Any feature that fetches a user-supplied URL (import, preview, webhook, image fetch) that has some SSRF protection in front of it.
- **Recon signal:** The endpoint fetches external resources and displays results, but an obvious internal address like 127.0.0.1 is rejected with an error. That rejection means a filter exists and can be probed.
- **Root cause:** Defenses are mostly blocklists of internal ranges (because the feature must fetch arbitrary external sites), compared against the input text rather than the final resolved address, and often ignore redirects, IPv6 or alternative encodings. Allowlists can be undermined by weak matching or by open redirects on allowed sites.
- **Exploitation steps:**
  1. Identify whether the protection looks like a blocklist or an allowlist from error behavior.
  2. For an allowlist: look for an open redirect on an allowed domain, or test weak matching (allowed name used as a subdomain or path of an attacker domain).
  3. For a blocklist: try redirect from your server, DNS pointing your hostname to an internal IP, IPv6 forms, and the encoding variants below.
  4. If stuck, design what you would write as the protection yourself and attack that design.
- **Bypasses used:**
  - Open redirect on an allowlisted domain.
  - Allowlist regex weakness (allowed domain embedded as subdomain or path of attacker domain).
  - Redirect via attacker-controlled server returning a 3xx to an internal address.
  - DNS record on attacker domain resolving to an internal IP (DNS pointing).
  - IPv6 addresses where only IPv4 was blocked.
  - Hex encoding of IP octets.
  - Octal encoding of IP octets.
  - Dword (single 32-bit decimal integer) encoding.
  - URL percent-encoding of the hostname (for example localhost).
  - Mixed encoding (combining hex, octal and decimal across octets).
- **Impact:** Not stated; enables reaching internal services that the filter was meant to block.
- **Bounty:** n/a (reference article).
- **Lesson / pattern:** Filters that compare text instead of the resolved, post-redirect destination fail; go through a checklist (redirect, DNS, IPv6, each encoding) and think like the developer who wrote the check.
- **Source:**
  - Title: Bypassing SSRF Protection (There's always more to do...)
  - Author: Vickie Li
  - URL: https://vickieli.medium.com/bypassing-ssrf-protection-e111ae70727b
  - Date: 2019-06-08 (page metadata says 2020-05-22)
  - Disclosed report: none
  - Raw file: `sources/ssrf/28-medium-vickie-li-bypassing-ssrf-protection.md`

Tags: filter-bypass, ip-obfuscation, redirect, dns-pointing, ipv6, allowlist-bypass
