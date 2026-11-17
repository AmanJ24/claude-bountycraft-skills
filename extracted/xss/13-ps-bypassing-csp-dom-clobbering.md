# Bypassing CSP via DOM clobbering (script.src gadget)

- **Vuln type / sub-variant:** DOM clobbering used to bypass a nonce + strict-dynamic CSP and reach a `script.src` sink. Requires HTML injection (not script injection).
- **Target type:** Library/app class: sites that have an HTML-injection hole, run a strict CSP (script-src with a nonce and `strict-dynamic`), and contain JS that reads a config-style variable to build a script URL. Modeled on a real bug-bounty-style codebase.
- **Recon signal:** Tester recognizes exploitability when: (1) they can inject HTML but injected `<script>` is blocked by CSP; and (2) a JS variable that controls a sink (here a dynamically created script's `src`) is looked up by name and is *not* otherwise defined, so a named HTML element can shadow it. DOM Invader flags this by reporting a `script.src` sink whose value contains the clobbered property names plus a canary.
- **Root cause (mechanism):** Named HTML elements are exposed on `document` and on `window` by their `id`/`name`. When JS reads a global/property (e.g. a `codeBasePath`-style config value) that is undefined, the browser's named-element lookup can resolve it to an attacker-planted element instead. The attacker thereby controls a variable the code trusts. Because the CSP uses `strict-dynamic`, a nonce-approved script is permitted to create further scripts, so a clobbered value that flows into a newly created `script.src` inherits trust and loads attacker-chosen code, defeating the host allowlist.
- **Exploitation steps (technique in plain steps):**
  1. Confirm HTML injection and identify the config property the code feeds into `script.src` (the writeup's test case used two names, an outer object and a `codeBasePath`-style sub-property).
  2. Clobber a multi-level property by combining two anchor elements: one sets the base `id`, the second reuses that `id` and adds a `name` plus the `href` that becomes the resolved value. Minimal illustrative markup: `<a id=ehy><a id=ehy name=codeBasePath href=...>` (two same-id `a` tags form a collection; the second's `name`/`href` supplies the nested value).
  3. Point `href` at the attacker script. A `data:` URL works but is not required; an HTTP URL to an attacker host works too. Account for a trailing path the code appends (e.g. `/utils.js`) by using a `//` single-line comment or a `?` so the appended filename lands in the query string.
  4. The strict-dynamic-trusted loader creates a script from the clobbered URL and executes attacker code despite CSP.
- **Bypasses used:** DOM clobbering of a JS variable lookup (named-element shadowing), abuse of `strict-dynamic` nonce propagation, path-tail handling via comment or query string.
- **Impact:** XSS on a page protected by a nonce-based strict CSP, i.e. code execution where script injection alone is blocked.
- **Bounty:** n/a (research/test case).
- **Lesson / pattern:** `strict-dynamic` plus a config variable resolved by name is a clobbering target: undefined globals can be overwritten by named HTML elements and flow into trusted sinks. Defenses: initialize config variables explicitly (never rely on an undefined global), avoid building `script.src` from DOM-reachable names, and use `Object`/namespacing that cannot be shadowed by element `id`/`name`. Clobbering needs only HTML injection, so CSP alone is not sufficient.
- **Source:**
  - Title: Bypassing CSP via DOM clobbering
  - Author: Gareth Heyes (PortSwigger Research)
  - URL: https://portswigger.net/research/bypassing-csp-via-dom-clobbering
  - Date: 2023-06-05
  - Disclosed report: none
  - Raw file: `sources/xss/13-ps-bypassing-csp-dom-clobbering.md`

Tags: dom-clobbering, csp-bypass, dom, gadget
