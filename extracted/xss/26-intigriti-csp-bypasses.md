# CSP bypasses: advanced exploitation guide (Intigriti)

Reference / technique writeup. A practical CSP-bypass methodology oriented at bug bounty. Note: several code examples (directive table, nonce code, CSP-injection code) were embedded as images and not captured; this note reconstructs the techniques from surviving prose and CSP strings.

- **Vuln type / sub-variant:** CSP bypass to enable XSS. Covers missing CSP, report-only mode, non-restrictive declarations (wildcard / `unsafe-inline`), allowlisted-CDN abuse, JSONP endpoints, file-upload-as-script, predictable nonces, and CSP injection.
- **Target type:** Any app that ships a CSP as its primary client-side defense, especially one that allowlists third-party CDNs/hosts or builds its policy dynamically from user-influenced input.
- **Recon signal:** Inspect the `Content-Security-Policy` header or the `<meta http-equiv>` tag. Flags: no CSP on some routes, `Content-Security-Policy-Report-Only` (monitored, not enforced), `script-src *` or `unsafe-inline`, allowlisted CDNs hosting vulnerable library versions, nonces that look timestamp/user-agent derived or reused, and a policy segment reflected from a request parameter.
- **Root cause:** CSP is frequently treated as the sole defense. Loose source lists, report-only deployments, trusting an entire CDN, predictable nonce generation, and dynamic policy construction from attacker-influenced input all leave cracks that re-enable script execution.
- **Exploitation steps (technique classes, defensively):**
  1. Missing / report-only CSP: no enforcement, so injected script runs unrestricted. Detect via headers.
  2. Non-restrictive source list: a `script-src *` allows loading an external script; `unsafe-inline` allows an injected inline handler (illustrative shape `'"><svg onload=...>`). Tools like CSP Evaluator flag these.
  3. Allowlisted-CDN abuse: a policy trusting popular CDNs can be bypassed by selecting a vulnerable library version already hosted there (tooling such as a CSP-bypass gadget finder helps locate a working source).
  4. JSONP-endpoint class: if any allowlisted host exposes a JSONP/callback endpoint, loading it as a script with a crafted callback reflects attacker JS from a trusted origin, satisfying `script-src`.
  5. File-upload-as-script: if the same host (or any allowlisted host) accepts arbitrary file uploads, a file whose bytes are valid JavaScript (the writeup cites CSPT research showing a crafted PDF can serve JS) can be loaded as a script source.
  6. Predictable nonce: when the nonce is derived from a universal timestamp plus client user-agent and rotated only every 60 seconds, the attacker can compute the current value and attach it to an injected script. Reused or static nonces are equally fatal.
  7. CSP injection / report-uri idea: when the policy is partly built from a request parameter (for example a `csp_report_uri`-style value), an attacker can inject an additional, more permissive directive to override the enforced policy. Illustrative shape: a report-uri parameter followed by `; script-src * 'unsafe-inline';` plus a script in a second reflected parameter. On older stacks, CR/LF injection can add response headers.
- **Bypasses used:** wildcard/`unsafe-inline` abuse, vulnerable CDN gadget, JSONP callback reflection, upload-as-script (polyglot file), predictable/reused nonce prediction, CSP directive injection / header injection.
- **Impact:** Arbitrary JavaScript execution via an XSS that CSP was supposed to block. The writeup stresses CSP bypasses are usually only accepted when chained to a real XSS.
- **Bounty:** n/a (technique article).
- **Lesson / pattern:** Do not rely on CSP alone. Pin exact CDN paths and versions, audit allowlisted hosts for JSONP and upload endpoints, generate long random per-request nonces (never derived from predictable inputs), and never build a CSP from unsanitized request input.
- **Source:**
  - Title: CSP Bypasses: Advanced Exploitation Guide (Bypassing Content Security Policy)
  - Author: Ayoub (Intigriti)
  - URL: https://www.intigriti.com/researchers/blog/hacking-tools/content-security-policy-csp-bypasses
  - Date: 2025-11-30 (last updated 2026-10-15)
  - Disclosed report: none
  - Raw file: `sources/xss/26-intigriti-csp-bypasses.md`
  - Note: some code examples were images, not captured.

Tags: csp-bypass, jsonp, nonce-reuse, polyglot, methodology
