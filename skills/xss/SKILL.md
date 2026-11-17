---
name: xss
description: Use when testing any place user input reaches a browser as HTML, JavaScript, a URL, or a client-side template (reflected/stored/DOM XSS), including hard cases that defeat modern defenses: sanitizer/mutation XSS (DOMPurify bypasses), CSP bypass, client-side template injection (AngularJS), DOM clobbering, prototype-pollution-to-XSS, postMessage DOM XSS, cookie-reflected and blind/stored XSS. Covers recognizing the sink context, choosing a vector that survives it, bypassing filters/WAF/CSP/sanitizers, escalating to account takeover, avoiding false positives, and reporting. Research/methodology only: it does not attack live targets.
---

# XSS (Cross-Site Scripting)

Synthesized from 30 real writeups and references: disclosed HackerOne reports (cookie-XSS to account takeover, blind-stored XSS firing in admin panels, postMessage DOM XSS across 100k+ sites), PortSwigger Research (DOMPurify mXSS bypasses, CSP bypass via DOM clobbering, AngularJS sandbox escapes, prototype-pollution gadgets), the PortSwigger XSS cheat sheet, OWASP, and expert practitioner blogs. Every claim traces to `sources.md`.

**This skill targets the hard cases on purpose.** Trivial reflected XSS in an unencoded search box barely exists on mature targets. Real findings today live in: the gaps modern frameworks leave (an `innerHTML`/`dangerouslySetInnerHTML` sink, a client-side template), sanitizer bypasses (mutation XSS), defense bypasses (CSP, WAF), and client-side data flows (DOM sources, postMessage, prototype pollution). The depth is in `references/`.

**Scope rule:** methodology only. It does not fire payloads at live targets. Real testing needs an authorized, in-scope program and your own test accounts/victim; prove execution with a harmless marker (a benign `print()`/`alert` or an out-of-band callback you control), never real cookie theft against real users (see `../../CLAUDE.md`).

## 1. Recognize it

XSS is present when attacker-controlled input is parsed by the victim's browser as code: HTML that creates an element/handler, JavaScript that runs, a `javascript:`/`data:` URL, or a client-side template expression that evaluates. The core skill is not "inject `<script>`", it is **identify the exact sink context** and pick a vector whose break-out characters and execution primitive survive that context and any filter.

Three delivery types:
- **Reflected:** input echoed in the immediate response (search, error, URL param, a reflected cookie).
- **Stored:** input persisted then rendered later, to you or to someone else (profile/name/comment, support form, file metadata, markdown, activity feed). **Blind** when it fires somewhere you cannot see, typically a staff/admin view.
- **DOM:** a client-side JS **source** (`location.hash`/`.search`, `document.cookie`, a `postMessage` `event.data`, `window.name`) flows to a dangerous **sink** (`innerHTML`, `document.write`, `eval`, `setTimeout`, `script.src`, jQuery `$()`) with no server round trip.

Modern tells that a real bug may be hiding:
- The app uses a sanitizer (DOMPurify) on rich text or markdown -> test **mutation XSS**.
- The page is an AngularJS/Vue template (`ng-app` in source) -> `{{7*7}}` -> **client-side template injection**.
- Client JS reads `location.hash`/a `postMessage` into a sink -> **DOM XSS**.
- A query/JSON parser accepts `__proto__` -> **prototype pollution** source; look for a gadget.
- A CSP header is present -> injection may be real but need a **CSP bypass** to execute.
- A cookie value is reflected, or a response is cacheable with user input in it -> **cookie-reflected / cache-poisoned** XSS.

## 2. Where it hides

Feature classes that produced findings. Full list with example reports in `references/attack-surfaces.md`.

- **Rich text / markdown / HTML editors behind a sanitizer** (DOMPurify): the bug is a sanitizer/mutation bypass, not plain injection (DOMPurify mXSS series).
- **Blind-stored sinks**: any free-text field read later by staff, support/contact forms, user name/company, tracker/campaign names, report fields, that render in an internal admin panel unencoded (Topcoder, DoD, Basecamp, Shopify). Fire with an out-of-band hook.
- **postMessage / web-message handlers**: cross-frame listeners that trust origin weakly or pass a message field to `innerHTML` (Lyst, Automattic/Jetpack across 100k+ sites).
- **DOM sources into sinks**: `location.hash`/`search` into `innerHTML`/`document.write`/jQuery `$()`; `document.cookie` reflected (Grammarly).
- **Client-side templates**: AngularJS/Vue expressions reflected (`{{ }}`), XSS even when HTML is encoded (AngularJS CSTI).
- **Prototype-pollution sources**: query/hash/JSON merged unsafely, reaching a gadget in app code or a shipped library (analytics, tag manager, jQuery, Lodash) that hits a sink.
- **Cookie-reflected / cache-poisoned**: a writable cookie reflected unescaped; an unkeyed input cached into a shared response (Grammarly, Abritel).
- **File uploads**: SVG (executes script when viewed same-origin), HTML, file names and metadata.
- **CSP-protected injection points**: where you have injection but need a CSP-bypass gadget (allowlisted JSONP/framework host, base-tag, DOM clobbering under strict-dynamic).

## 3. Test methodology

Short version. Full step-by-step in `references/contexts-and-payloads.md` and `references/advanced-techniques.md`.

1. **Find the flow.** Locate every reflection, stored field, and DOM source->sink. For DOM, read the JS (or use DOM Invader) to trace source to sink.
2. **Identify the exact context.** Send a unique canary (e.g. `qzx1'">` plus a tag) and view where it lands and how it is transformed: HTML body, quoted/unquoted attribute, URL attribute, JS string/comment/template literal, or a framework template. The context dictates the vector.
3. **Confirm execution with a context-appropriate probe,** not a reflexive `<script>`: break out of the attribute/string, or use an event handler (`onfocus`+`autofocus`), or `{{7*7}}` for a template. Use a benign marker (`print()`, `alert(document.domain)`) or an OOB callback.
4. **If a sanitizer is in the way,** test mutation XSS (namespace confusion with `math`/`svg`/`style`, node smuggling, clobbering the sanitizer's own state). See `references/advanced-techniques.md`.
5. **If it is blind,** deliver an out-of-band hook (XSS Hunter / interactsh / your collaborator) and spray multiple fields; wait for the admin-side callback.
6. **If a CSP blocks execution,** read the policy (CSP Evaluator) and find a bypass gadget: allowlisted JSONP/framework host, missing `base-uri`, DOM clobbering under `strict-dynamic`, a readable nonce. See `references/csp-and-waf-bypass.md`.
7. **If a WAF/filter strips payloads,** switch context break-out, event handler, encoding, no-parens/no-`>` forms, or stash the payload in `window.name`. See `references/csp-and-waf-bypass.md`.
8. **Escalate to impact, then stop at proof.** Demonstrate execution in the target origin; show the escalation path (session/cookie theft, forced login, admin takeover) with your own test victim. Do not steal real users' data.

## 4. Known bypass patterns

Summary; full catalogues in `references/`.

- **Filter / WAF** (`references/csp-and-waf-bypass.md`): break out via the real context, alternative event handlers and tags (`onfocus`+`autofocus`, `ontoggle`, `onerror`), case/encoding (HTML entities, UTF-7, overlong UTF-8, JS unicode escapes), keyword-splitting (`self['ale'+'rt']`), no-parens (`onerror=alert;throw 1`, `` alert`1` ``), no-`>` (`<svg onload=alert(1)//`), length-limited vectors, and `window.name` payload smuggling.
- **Sanitizer / mutation XSS** (`references/advanced-techniques.md`): namespace confusion across HTML/SVG/MathML (`math`/`mtext`/`mglyph`/`style`) so sanitized-safe markup re-parses into a live `img onerror`; smuggle the mutation in comment/CDATA nodes; DOM-clobber the sanitizer's own guard state (`parentNode`, `__depth`); exploit double/triple re-parse (sanitize-twice apps). Core principle: `parse(serialize(tree)) != tree`.
- **CSP** (`references/csp-and-waf-bypass.md`): allowlisted JSONP endpoint, allowlisted host serving a framework (AngularJS via reCAPTCHA) as a gadget, base-tag relative-path hijack when `base-uri` is unset, DOM clobbering into `script.src` under `strict-dynamic`, read the live nonce from the DOM (`node.nonce`) then inject a nonce'd script, open-redirect/RPO path bypass.
- **Framework** (`references/advanced-techniques.md`): AngularJS CSTI with a version-matched sandbox escape (`{{constructor.constructor('alert(1)')()}}`, the `charAt=[].join` family), and client-side prototype-pollution gadgets (fetch options, `Object.defineProperty`, storage, GA `hitCallback`->`setTimeout`, GTM `sequence`->`eval`, jQuery/Lodash) reached from a `__proto__` source.

## 5. Common false positives

- **Self-XSS with no delivery.** A payload that only runs in your own session (you paste it into your own devtools/field) is not a finding without a cross-user delivery vector (a cookie-write primitive, a stored/blind sink, a link that carries it). The Grammarly bug was only real once a cross-site cookie-write existed.
- **Framework auto-encoding.** React/Angular/Vue encode interpolated text by default; a reflected value that renders as inert text is not XSS. The bug is in the escape hatch (`dangerouslySetInnerHTML`, `v-html`, `innerHTML`, a template expression), not the encoded output.
- **CSP actually blocks it.** If injection is real but no CSP bypass works and nothing executes, it is reduced severity (HTML injection / dangling-markup at most), not XSS. Say so honestly.
- **Encoded / non-executing context.** Value reflected inside a context that cannot execute (a correctly-encoded text node, an attribute that is not an event/URL sink) is not XSS.
- **A sanitizer that holds.** Rich text that DOMPurify renders inert, with no working mutation, is not a finding; the finding is a specific bypass.
- **DOM "source" that is not attacker-controllable,** or a sink fed only by trusted data.
- **`alert(1)` as the whole report.** Execution is necessary but the report needs context, reach, and impact, not just a popup.

## 6. Reporting

What made reports in this set land (template in `references/escalation-and-reporting.md`):
- **Prove execution in the target origin** with a harmless marker, and name the exact context and sink and the defense that failed (unescaped `<noscript>`, `innerHTML` in a postMessage handler, a sanitizer mutation, a missing `base-uri`).
- **Escalate and show reach.** Cookie/session theft to account takeover, forced login, or (for blind) firing in an admin panel and capturing the admin session. Reach matters: stored/blind hits everyone who views it; a postMessage bug on a shared widget origin hits every consuming site (Jetpack: 100k+ domains).
- **Show delivery** for anything that looks like self-XSS: the link, the cookie-write, the stored field.
- **Redact real secrets and use test accounts.** Capture one stolen test cookie or one admin-callback screenshot, not a real victim's data.
- **State the impact rung and blast radius**, and tie to concrete loss (account takeover, admin compromise, mass client-side control).

## 7. Sources

See `sources.md` (title, author, URL, disclosed report, bounty) and `../../extracted/xss/` for the per-writeup notes. Deeper references:
- `references/attack-surfaces.md`: feature classes with example reports
- `references/contexts-and-payloads.md`: the context model, representative vectors per context, polyglots, DOM sources/sinks
- `references/advanced-techniques.md`: mutation XSS, DOM clobbering, prototype pollution, AngularJS CSTI, postMessage (the expert core)
- `references/csp-and-waf-bypass.md`: CSP bypass classes and WAF/filter evasion
- `references/escalation-and-reporting.md`: weaponization, blind-XSS OOB, account-takeover chains, write-up template
