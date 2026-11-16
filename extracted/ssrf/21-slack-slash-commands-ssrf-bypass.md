# Slack slash command URL: IPv6 unspecified-address redirect bypasses SSRF protection

- **Vuln type / sub-variant:** Semi-blind SSRF (error/response visible per command) with port probing, via redirect to an IPv6 zero address; bypass of fixes from two earlier reports.
- **Target type:** Developer platform feature: custom slash command whose request URL is a hunter-chosen domain that Slack's servers call when the command is typed.
- **Recon signal:** Slack servers make an outbound call to an attacker-set URL, and prior reports (#61312 and #356765) on the same feature meant there were existing filters worth retesting.
- **Root cause:** The protection checked the initial URL against internal addresses but not the redirect target, and did not account for the IPv6 unspecified address, which maps to the local host.
- **Exploitation steps:**
  1. Create a Slack app with a slash command and set its request URL to a domain the hunter owns.
  2. On that server, return a redirect whose Location points to the bracketed IPv6 zero address with an internal port (SSH port used).
  3. Run the slash command in Slack.
  4. Observe the difference in resulting response or error for different ports; the report shows results for SSH and SMTP targets.
- **Bypasses used:** Redirect via attacker-controlled 302 combined with IPv6 address form ([::] instead of 127.0.0.1) to evade the blocklist.
- **Impact:** Internal port scanning and service reachability from Slack's infrastructure. Rated medium.
- **Bounty:** awarded, amount hidden on page (not stated in source).
- **Lesson / pattern:** After a fix, retest the same feature with IPv6 forms and a redirect hop; blocklists written for IPv4 literals often miss [::], [::1] and mapped addresses, and redirect targets are frequently validated less strictly.
- **Source:**
  - Title: SSRF in api.slack.com, using slash commands and bypassing the protections
  - Author: elber
  - URL: https://hackerone.com/reports/381129
  - Date: 2019-02-22 (disclosed; reported 2018-07-13)
  - Disclosed report: https://hackerone.com/reports/381129
  - Raw file: `sources/ssrf/21-slack-slash-commands-ssrf-bypass.md`

Tags: webhook, redirect, ipv6, filter-bypass, port-scan, patch-bypass
