# CSP bypass on PortSwigger.net via Google script resources (joaxcar)

Real disclosed finding (defensive disclosure). A purely-CSP-bypass report accepted by PortSwigger. Demonstrates the allowlisted-trusted-script class of bypass and the escalation from a restricted alert to arbitrary XSS by reading the live nonce.

- **Vuln type / sub-variant:** CSP bypass of a nonce-based `script-src` that also carries a URL allowlist but lacks `strict-dynamic`. The allowlist includes a Google reCAPTCHA host that serves an AngularJS-bearing script, which becomes the bypass gadget.
- **Target type:** A mature site (portswigger.net) using a nonce plus an allowlist for `script-src`, with no HTML-injection point required for the report itself (reported as a standalone CSP weakness).
- **Recon signal:** Reading the target's CSP in CSP Evaluator showed a `nonce` but no `strict-dynamic`, which means allowlisted URLs remain valid script sources. The allowlist contained `https://www.google.com/recaptcha` and `https://www.gstatic.com/recaptcha`, both of which host `about/js/main.min.js` containing AngularJS. The same setup had been seen on Twitter's CSP from a Google CTF writeup.
- **Root cause:** Without `strict-dynamic`, a nonce policy still trusts every URL in the allowlist. An allowlisted host serving a client-side template framework (AngularJS, the classic CSP breaker) lets an attacker achieve templating-based execution from a trusted origin. Secondary issue: the policy also omitted the `form-action` directive.
- **Exploitation steps (mechanism, defensively):**
  1. Load the allowlisted Google reCAPTCHA script (which bundles AngularJS) as an allowed source, then introduce an Angular template expression via an element attribute error handler to trigger a restricted execution (an alert). Minimal illustrative gadget shape: `<img src=x ng-on-error='...alert(1)...'>` after the allowlisted AngularJS script is present.
  2. Restricted-to-full escalation: the policy lacked `unsafe-eval`, so string-eval was blocked and the alert alone was not arbitrary code execution. The author escalated by reading the page's still-accessible nonce from the DOM. Although nonce values are hidden from CSS side channels, they remain readable from JavaScript via `document.querySelector("[nonce]").nonce`.
  3. With the live nonce in hand, the Angular-driven code creates a fresh `<script>` element, sets its `src` to an attacker URL, assigns the stolen nonce, and appends it, yielding a valid nonce'd script that loads arbitrary JS: a full CSP bypass.
  4. On PortSwigger.net the identical Twitter-style setup was present, so the same technique applied. The author had no HTML-injection sink to chain, so reported the CSP bypass itself, citing PortSwigger's own prior Piwik PRO CSP-bypass research as evidence the program valued such reports.
- **Bypasses used:** allowlisted-trusted-script (AngularJS via reCAPTCHA) gadget, nonce exfiltration from the DOM via `node.nonce`, dynamic nonce'd script injection; note the inverse trusted-script pattern (jQuery adding a nonce'd script under `strict-dynamic`) referenced as prior art.
- **Impact:** Full CSP bypass enabling arbitrary script execution context on portswigger.net had an HTML-injection sink existed; separately, the missing `form-action` directive could allow credential leaks via form hijacking.
- **Bounty:** $1000 for the CSP bypass plus a $500 bonus for the missing `form-action` issue.
- **Lesson / pattern:** A nonce without `strict-dynamic` still trusts the whole URL allowlist, and any allowlisted host serving AngularJS (or similar template framework) is a bypass gadget. Nonces are readable from JS, so a single trusted-script foothold escalates to arbitrary execution. Harden with `strict-dynamic`, drop framework-bearing third-party hosts from `script-src`, and set `form-action`.
- **Source:**
  - Title: CSP bypass on PortSwigger.net using Google script resources
  - Author: Johan Carlsson (joaxcar)
  - URL: https://joaxcar.com/blog/2024/02/19/csp-bypass-on-portswigger-net-using-google-script-resources/
  - Date: 2024-02-19
  - Disclosed report: https://hackerone.com/reports/2279346
  - Raw file: `sources/xss/27-joaxcar-csp-bypass-portswigger.md`

Tags: csp-bypass, nonce-reuse, gadget, methodology
