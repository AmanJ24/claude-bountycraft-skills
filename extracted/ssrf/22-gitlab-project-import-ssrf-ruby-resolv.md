# GitLab import-by-URL SSRF filter bypass via Ruby Resolv (CVE-2017-0904)

- **Vuln type / sub-variant:** SSRF blocklist bypass in an import-from-URL feature, giving internal port scanning via git clone errors.
- **Target type:** Project import from a remote repository URL on GitLab.com (and self-hosted GitLab CE/EE).
- **Recon signal:** An earlier report on the same feature had been fixed by blocking 127.0.0.1 and localhost, so the hunter retested with alternate loopback notations. Error messages from the clone attempt revealed connection outcomes.
- **Root cause:** GitLab's URL blocker called Resolv.getaddresses on the hostname and compared the results to a blocked IP list. For many non-canonical spellings Ruby's resolver returns an empty list, so no blocked IP matched, while git/curl later let the OS turn the same string into 127.0.0.1 (inet_aton style parsing). Outcomes also varied by machine and Ruby version, which initially made GitLab unable to reproduce. The dev team also noted their original design allowed hosts that did not resolve, because blocking all unresolved names would break projects saved while DNS was down.
- **Exploitation steps:**
  1. Submit an import URL with loopback in octal or hex dotted form plus a port (shapes like 0177.1:22 or 0x7f.1:22).
  2. Read the clone error text: a reset connection or banner-related failure versus a plain refusal tells open from closed ports.
  3. Show further variants: a single 32-bit decimal integer, zero-padded octets, and a bare zero.
  4. Reproduce locally with a small script that shows the resolver returns nothing while the HTTP client still connects to a local listener, proving the mismatch.
  5. Propose switching to Socket/Addrinfo getaddrinfo (what the OS actually uses).
- **Bypasses used:** Resolver parsing quirk (Ruby Resolv returns an empty array), IP notation obfuscation (octal, hex, decimal dword, zero-padded, bare 0). Also considered DNS TOCTOU but not demonstrated in source.
- **Impact:** Internal network port scanning from GitLab.com's servers; severity medium (5.4). Fixed in GitLab 10.1.2 by resolving through Addrinfo and requiring resolution. A CVE was assigned to the underlying Ruby behavior and other SSRF gems were also affected.
- **Bounty:** none (stated on report page).
- **Lesson / pattern:** Same Ruby Resolv root cause as the HackerOne Integrations finding in note 16; when a blocklist runs in one resolver and the connection runs in another, any string they parse differently is a bypass, so enumerate notations and test on the real stack, and fail closed on empty resolution.
- **Source:**
  - Title: SSRF vulnerability in gitlab.com via project import (plus author advisory on the Ruby Resolv bug, extra source for root-cause detail, row 30)
  - Author: edoverflow
  - URL: https://hackerone.com/reports/215105 ; https://edoverflow.com/2017/ruby-resolv-bug/
  - Date: reported 2017-03-21, disclosed 2017-11-09
  - Disclosed report: https://hackerone.com/reports/215105
  - Raw file: `sources/ssrf/22-gitlab-project-import-ssrf-ruby-resolv.md` ; `sources/ssrf/30-edoverflow-ruby-resolv-bug.md`

Extra detail from the advisory: the empty result is caused by the library doing its own DNS-style lookup and hosts file reading rather than asking the OS; some other Ruby SSRF gems shared the flaw and were patched or already safe. The advisory also shows the GitLab flow: the earlier fix relied on a blocklist applied to Resolv output, which is why simple notation variants defeated it.

Tags: import-from-url, filter-bypass, ip-obfuscation, resolver-quirk, ruby, port-scan
