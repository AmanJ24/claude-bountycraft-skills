# XSS attack surfaces (with example findings)

Each surface: what it is, the recon signal, how execution happens, and the source notes in `../../extracted/xss/`. Use it to decide where to look and which technique applies.

## Rich text / markdown / HTML editors behind a sanitizer
The target accepts HTML (comments, descriptions, wiki, markdown, email) and runs it through a sanitizer (usually DOMPurify) before `innerHTML`. Plain injection fails; the bug is a **sanitizer/mutation bypass**.
- DOMPurify SVG/MathML mutation XSS (notes 09, 12, 20, 21, 22, 23). The vector is namespace confusion (`math`/`mtext`/`mglyph`/`style`, SVG `<style>`) that re-parses into a live `img onerror`.
- **Recon:** find where your HTML is sanitized then inserted; identify the DOMPurify version; test mutation payloads (see `advanced-techniques.md`). Apps that sanitize twice (e.g. Mermaid) enable triple-parse bypasses.

## Blind-stored sinks (fire in an admin/staff view)
Any free-text field persisted and later rendered to staff unencoded. You never see it fire; an out-of-band hook does.
- Topcoder contact form -> admin CRM panel, leaked admin session + PII (note 05).
- DoD profile fields -> admin view (note 06); Basecamp cross-account stored XSS -> admin (note 04); Shopify activity -> admin cookies (note 03); tracker/campaign names -> admin stats pages (candidate index).
- **Recon:** name, company, description, subject, tracker/campaign/report names, anything support or admins read. Spray a blind hook (XSS Hunter/interactsh) across every field; wait for the admin-side callback. Back-office views are often unescaped even when public pages are not.

## postMessage / web-message handlers
A cross-frame `message` listener that validates origin weakly (missing, naive `includes`/`indexOf`, null-origin) or passes a message field to a sink.
- Lyst DOM XSS via insecure postMessage handler (note 07).
- Automattic/Jetpack: `avatar_URL` from a postMessage into `innerHTML`; origin check trusted widgets.wp.com, which itself had a DOM XSS, so the allowlist was defeated; fired on every Likes-enabled site, 100k+ domains (note 08).
- **Recon:** enumerate `addEventListener('message', ...)` handlers (DOM Invader web-message mode). Check the origin check and whether `event.data` fields reach `innerHTML`/`eval`/`document.write`. If origin is allowlisted, look for XSS on the allowed origin.

## DOM sources into sinks (no server round trip)
A client JS source flows to a sink.
- Sources: `location.hash`, `location.search`, `document.URL`, `document.referrer`, `document.cookie`, `window.name`, `postMessage` data.
- Sinks: `innerHTML`/`outerHTML`, `document.write`, `eval`/`Function`, `setTimeout`/`setInterval` (string), `script.src`/`iframe.src`, jQuery `$()`/`.html()`/`.append()`, `location`/`location.href`.
- Grammarly `document.cookie` reflected into `<noscript>` HTML (note 01).
- **Recon:** grep the JS for sinks, trace back to a source; DOM Invader automates this.

## Client-side template injection (framework templates)
The page is an AngularJS (or Vue) template; a reflected `{{expression}}` evaluates even when HTML is encoded.
- AngularJS CSTI + sandbox escape (notes 14, 15, 17). `{{7*7}}` -> `49` confirms.
- **Recon:** `ng-app` in view-source above the reflection; `{{7*7}}`/`{{1+1}}` probe. HTML-encoding does not protect; the sandbox is not a boundary.

## Prototype-pollution sources + gadgets
A query/hash/JSON parser or deep-merge lets `__proto__[x]` write `Object.prototype.x`; later code reads an unset property into a sink.
- Widespread gadgets in browser APIs and shipped libraries (note 16): fetch options, `Object.defineProperty`, storage, GA `hitCallback`->`setTimeout`, GTM `sequence`->`eval`, Adobe DTM, jQuery/Lodash/sanitize-html (cheat-sheet table, note 17). Real-world chains on Mozilla/Jira/Apple (note 28).
- **Recon:** `?__proto__[foo]=bar` then check `Object.prototype.foo` in console; DOM Invader prototype-pollution mode finds source+gadget.

## Cookie-reflected / cache-poisoned
- A cookie value reflected unescaped, combined with a way to set that cookie cross-site, is real XSS (Grammarly, note 01).
- An unkeyed header/cookie reflected into a response the cache stores and serves to others turns reflected into stored (Abritel `hav` cookie cached into a `.js`-extension page; session was readable in a JS variable, note 02).
- **Recon:** reflected cookies; cacheable responses (`.js`/`.css` extensions, `Cache-Control`) that contain user input; a session value exposed in a JS global.

## File uploads
- SVG with a `<script>`/handler executes when viewed on the same origin; HTML uploads; XSS via file name/metadata.
- **Recon:** upload an SVG/HTML and view it on an origin that matters; check content-type and sandboxing.

## CSP-protected injection points
You have HTML injection but a CSP blocks inline/script execution; you need a CSP-bypass gadget.
- joaxcar: allowlisted reCAPTCHA host serves AngularJS as a gadget; nonce read from the DOM (note 27). CSP bypass via DOM clobbering into `script.src` under `strict-dynamic` (note 13). JSONP/base-tag/allowlist classes (notes 24, 26).
- **Recon:** read the CSP (CSP Evaluator); classify the weakness (allowlisted framework/JSONP host, missing `base-uri`, no `strict-dynamic`, clobberable config var). See `csp-and-waf-bypass.md`.

## Parameter / sink quick-list
Reflected/stored params: `q, search, s, query, name, title, comment, message, description, subject, company, url, redirect, returnUrl, lang, theme, callback, ref, utm_*, error, msg`.
DOM sinks: `innerHTML, outerHTML, insertAdjacentHTML, document.write(ln), eval, Function, setTimeout/Interval(str), script.src, iframe.src/srcdoc, a.href, location(.href), jQuery $()/.html()/.append()/.before()/.after()`.
DOM sources: `location.hash/.search/.href, document.URL/.documentURI/.referrer/.cookie, window.name, history.state, postMessage event.data, localStorage/sessionStorage`.
