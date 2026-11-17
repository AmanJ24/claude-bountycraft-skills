# Lyst: DOM XSS via insecure postMessage handler (no origin check), innerHTML sink

Source completeness: partial (short body; rated Low by the team, no cookie/session escalation demonstrated, only alert PoC).

- **Vuln type / sub-variant:** DOM-based XSS via web messaging (postMessage). An unauthenticated `message` event handler writes a message field to `innerHTML` with no origin validation.
- **Target type:** A reveal.js slide-deck notes page (talks.lystit.com .../notes.html) that registers a `postMessage` listener.
- **Recon signal:** The page registered a `window.addEventListener('message', ...)` handler. The hunter inspected the handler and saw it parsed `event.data` as JSON and passed a field straight into `innerHTML` with no `event.origin` check, the classic insecure-postMessage pattern.
- **Root cause:** The handler takes `event.data`, JSON-parses it, and assigns `data.notes` (or `marked(data.notes)`) to `notes.innerHTML`. Message field: `notes` (and `markdown` flag). Sink: `notes.innerHTML`. Failed defense: no `event.origin` allowlist, so any page can frame or open the target and post to it; no sanitization of the `notes` value before `innerHTML`.
- **Exploitation steps:**
  1. Host an attacker page that opens (or frames) the vulnerable notes page.
  2. Send a `postMessage` with JSON containing a `notes` field holding HTML with an auto-firing element (for example an image with an error handler).
  3. The handler assigns it to `innerHTML` and the script executes in the talks.lystit.com origin.
- **Bypasses used:** None needed; the origin check was absent entirely.
- **Impact:** Arbitrary JS in the victim's session on the target origin (proof was `alert`). Team rated it Low; no session/cookie escalation shown.
- **Bounty:** Awarded, amount hidden. Severity Low.
- **Lesson / pattern:** A `message` listener that reaches `innerHTML` without an `event.origin` check is cross-origin-triggerable DOM XSS; always validate the sender origin and sanitize before any HTML sink.
- **Source:**
  - Title: DOM XSS on talks.lystit.com
  - Author: gamer7112
  - URL: https://hackerone.com/reports/1031644
  - Date: 2021-02-09 (disclosed)
  - Disclosed report: https://hackerone.com/reports/1031644
  - Raw file: `sources/xss/07-lyst-dom-xss-postmessage.md`

Tags: dom, postmessage
