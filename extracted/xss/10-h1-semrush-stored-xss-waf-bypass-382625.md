# Semrush stored XSS with WAF bypass via client-side-only validation (H1 #382625)

- **Vuln type / sub-variant:** Stored XSS, reached by bypassing a blocklist WAF. Sink: a competitor-domain field in the Position Tracking / Rankings Distribution feature that is reflected back into HTML when the page renders.
- **Target type:** Authenticated SaaS web app feature (SEO position-tracking project), domain input field.
- **Recon signal:** The domain field was validated only on the client side. Tester intercepted the request in Burp, swapped the validated value for a payload, and saw it saved intact, classic signal that server-side storage trusts client-validated input. On render the payload was stripped by a WAF, which told the tester a server-side blocklist stood between storage and output.
- **Root cause:** Two layers failed. (1) Validation was client-side only, so arbitrary values could be stored. (2) The output-time defense was a blocklist WAF that filtered many tags/attributes but not all event-bearing constructs, and (per the thread) applied inconsistently across HTTP methods. No contextual output encoding of the stored value.
- **Exploitation steps:**
  1. Set up a Position Tracking project, open Rankings Distribution, edit the competitor list.
  2. Submit a valid domain (e.g. google.com), intercept in Burp, replace the domain parameter with an XSS payload.
  3. Use a construct the blocklist missed: an uncommon tag/attribute pairing such as a `marquee` element carrying an `onstart` handler (illustrative: `"><u>..</u><marquee onstart='alert(document.cookie)'>`), which evades the filtered-tag list.
  4. Save; close the settings page. The stored value renders and the handler fires.
- **Bypasses used:** WAF blocklist evasion via a rarely-filtered tag/event (`marquee`/`onstart`) instead of common `script`/`img`/`onerror`; the thread also notes a newline-splitting trick (splitting the payload across fields / onto a new line) defeated the rule, showing the WAF keyed on single-line patterns per method.
- **Impact:** Session/cookie theft and page defacement in the context of other Semrush users who view the tracked data. Separately, the WAF bypass itself was treated as an additional weakness that could aid other attacks.
- **Bounty:** Awarded, plus an explicit bonus for the WAF bypass (amount hidden by HackerOne). Severity settled at Medium (6.5).
- **Lesson / pattern:** Client-side validation is not a storage control, and a blocklist WAF is not output encoding. When a field is stored despite UI validation, test for a rare-tag/rare-event payload and for line/method-splitting to defeat pattern-based WAF rules. Real fix is contextual output encoding at the sink, not more blocklist rules.
- **Source:**
  - Title: Stored XSS in Position Tracking and WAF Bypass (Semrush, Report #382625)
  - Author: jimgogogo
  - URL: https://hackerone.com/reports/382625
  - Date: 2018-07-17 (disclosed 2018-12-07)
  - Disclosed report: https://hackerone.com/reports/382625
  - Raw file: `sources/xss/10-h1-semrush-stored-xss-waf-bypass-382625.md`

Tags: stored, waf-bypass, cookie-theft, reflected
