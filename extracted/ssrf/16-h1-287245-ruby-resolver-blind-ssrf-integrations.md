# Ruby Resolv empty-result bypass of SSRF filter in HackerOne Integrations

- **Vuln type / sub-variant:** Blind SSRF filter bypass (blocklist defeated by a resolver quirk); no data exfiltrated.
- **Target type:** Issue-tracker integration settings (Jira, Phabricator style) where a program owner supplies a server URL that HackerOne's backend contacts.
- **Recon signal:** The integrations panel makes the server fetch a user-supplied URL and a Ruby SSRF helper gem (private_address_check) was known to guard it. The hunter had just found that Ruby's Resolv library behaves differently across Linux machines for odd IP spellings, so a gem built on it was an obvious next victim.
- **Root cause:** The gem resolved the hostname with Resolv.getaddresses, then checked whether any returned IP was private. For some non-canonical IP spellings Resolv returns an empty list (its reverse DNS style lookup fails, it never defers to the OS). An empty list contains no private IP, so the check passes. The HTTP client then hands the original string to the OS, which happily interprets it as loopback. The filter and the connector disagree on what the string means, and "resolves to nothing" was treated as "safe".
- **Exploitation steps:**
  1. Identify that the integration URL is filtered by a gem that uses Resolv.getaddresses.
  2. Submit loopback written in shortened, octal, hex or zero-padded form with a port (shapes like 0177.1, 0x7f.1, 127.000.001 plus :22).
  3. Observe that the filter no longer rejects the URL, proving the bypass.
  4. Try to turn it into a real request; the reporter could not extract anything useful, so it stayed a bypass only.
- **Bypasses used:** Resolver parsing quirk (Ruby Resolv returns [] for IPv4 shorthand/octal/hex/padded forms on some OS setups) combined with IP notation obfuscation (octal, hex, zero-padded). Behavior varied per machine, which made it hard to reproduce.
- **Impact:** Potential internal network scanning from HackerOne's servers; actually demonstrated only as a filter bypass, with no exploitation shown. Triaged as worth fixing.
- **Bounty:** not stated in source.
- **Lesson / pattern:** Treat an empty DNS result as a failure, not as "no private IPs", and validate with the same resolver call the HTTP client uses (OS getaddrinfo); this is the same Ruby Resolv root cause as the GitLab project import finding in note 22, so test any Ruby SSRF filter with notation variants and compare against what the OS resolves.
- **Source:**
  - Title: Blind SSRF in 'Integrations' by abusing a bug in Ruby's native resolver (plus author advisory "Bypassing Server-Side Request Forgery filters by abusing a bug in Ruby's native resolver", extra source for root-cause detail, row 30)
  - Author: edoverflow
  - URL: https://hackerone.com/reports/287245 ; https://edoverflow.com/2017/ruby-resolv-bug/
  - Date: 2017-11-09
  - Disclosed report: https://hackerone.com/reports/287245
  - Raw file: `sources/ssrf/16-h1-287245-ruby-resolver-blind-ssrf-integrations.md` ; `sources/ssrf/30-edoverflow-ruby-resolv-bug.md`

Extra detail from the advisory (CVE-2017-0904): the library ignores the OS resolver and uses its own DNS and hosts file lookups, so any address the OS would accept via inet_aton but DNS cannot reverse-map returns an empty array. Another gem in the ecosystem (private_address_check) had the same flaw and was patched; ssrf_filter was safe because it errors on an empty result, and faraday-restrict-ip-addresses was safe because it uses Addrinfo.getaddrinfo. Recommended fix: resolve via the socket library (Addrinfo/Socket getaddrinfo) and fail closed on empty results.

Tags: blind, filter-bypass, ip-obfuscation, resolver-quirk, ruby, integrations
