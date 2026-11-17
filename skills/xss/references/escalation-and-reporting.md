# XSS escalation, weaponization, and reporting

`alert(1)` proves execution; it is not the finding. What moves severity and bounty is the escalation and the reach. Do this only against your own test accounts/victim in authorized scope, and prove it with a benign marker or one test-account credential, never a real user's data. Sources: notes 01, 02, 04, 05, 06, 08.

## Escalation paths (in rough order of impact)

**Session / cookie theft to account takeover.**
- If the session cookie is not HttpOnly, read `document.cookie` and exfiltrate it to a host you control, then replay.
- If it IS HttpOnly, look for a sibling endpoint that returns cookie values to a same-origin credentialed request (Grammarly: `/cookies` endpoint handed back the HttpOnly session to a same-origin XHR, defeating HttpOnly, note 01). Or read a session value exposed in a JS variable / cacheable page (Abritel: `window.INITIAL_STATE...cookie`, note 02).
- Prove with your own test victim's cookie; show it grants access to that account's data.

**Forced login / session fixation.**
- A cross-site cookie-write primitive plus XSS can overwrite the victim's session with the attacker's (forces the victim into an attacker-controlled account) or plant the payload cookie (Grammarly, note 01).

**Act as the victim via same-origin requests.**
- From the XSS origin, make credentialed requests to change email/password, read private data, exfiltrate a CSRF token, or perform privileged actions. This is often higher-signal and cleaner than cookie theft (no secret leaves the page).

**Blind / stored XSS firing in an admin panel (note 05, 06, 04).**
- Deliver an out-of-band hook (XSS Hunter / interactsh / your collaborator) in fields staff read; when an admin opens it, capture the admin session, internal URLs, DOM, and PII. This escalates a public contact form to admin account takeover.
- Spray multiple fields (name/company/subject/description) to find which reaches the unescaped admin sink.

**Mass / cross-consumer reach (note 08).**
- A stored XSS hits everyone who views the content. A postMessage/DOM bug on a shared widget origin hits every site embedding it (Jetpack Likes: 100k+ domains from one XSS on widgets.wp.com). State the blast radius explicitly; it is what rated these High.

## Weaponization notes from the set
- **Split the script tag** so the surrounding page keeps rendering while your external script loads (Grammarly).
- **Host over HTTPS** so an HTTPS target does not block your script/request as mixed content.
- **Chain two weaker bugs:** a cookie-write primitive (dismissed alone as "normal") + a cookie reflection = real XSS (Grammarly); a DOM XSS on a trusted origin + a weak origin check = postMessage XSS everywhere (Jetpack).
- **Cache-poison** a reflected XSS into a stored one by getting it into a cacheable response served to others (Abritel).

## Reporting template

**Title:** `<Reflected/Stored/DOM> XSS in <feature> via <sink/context> leading to <impact>`
e.g. "Stored XSS in support form fires in admin panel, leading to admin session theft".

**Summary:** where the input enters, the exact sink and context, the defense that failed, and the impact reached.

**Steps to reproduce:**
1. The injection point and the exact payload (benign marker), with the context it breaks out of.
2. Any bypass used (sanitizer mutation, CSP gadget, WAF evasion, origin-check bypass) stated plainly.
3. For DOM/postMessage: the source, the listener/origin logic, and the sink.
4. For blind: the field(s) sprayed and the out-of-band callback evidence.
5. The escalation: the request/flow that steals the (test) session or performs the privileged action, with the result.

**Root cause:** name it precisely: "cookie value reflected unescaped into a `<noscript>` block", "`avatar_URL` from a postMessage assigned to `innerHTML` with an origin check defeated by XSS on the allowed origin", "DOMPurify mutation via MathML namespace confusion", "nonce without `strict-dynamic` lets an allowlisted AngularJS host run code".

**Impact and reach:** the rung (execution -> data read -> session theft -> account/admin takeover) and the blast radius (one user / everyone who views stored content / every site embedding the widget). Suggest a CVSS with justification.

**Remediation:** context-correct output encoding at every sink; a trusted-types / strict CSP with `strict-dynamic` and no framework/JSONP hosts; a current sanitizer used as `RETURN_DOM` (no re-serialize-reparse); exact `event.origin` allowlists plus message-field encoding; HttpOnly + no sibling endpoint that returns cookie values; null-prototype config objects.

**Scope note:** confirm in scope; state you used a benign marker / test accounts and stopped at proof.

## What to avoid in a report
- Reporting self-XSS without a delivery vector.
- Submitting a bare `alert(1)` with no context, reach, or impact.
- Claiming XSS where a framework auto-encoded the value or the CSP actually blocked execution with no working bypass.
- Stealing or displaying a real user's cookie/data; use a test victim and redact.
