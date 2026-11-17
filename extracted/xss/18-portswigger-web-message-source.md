# DOM XSS via postMessage (web message source)

- **Vuln type / sub-variant:** DOM-based XSS where `event.data` from a `message` event is the source and flows to a sink (`eval`, `innerHTML`, `document.write`, `location`). Technique page, scraped in full.
- **Target type:** Any page with `window.addEventListener('message', ...)`: embedded widgets, parent/child iframe integrations, SSO or payment popups.
- **Recon signal:** A `message` listener in page JS (grep for `addEventListener('message'`), especially one that passes `e.data` to a sink or has a weak `e.origin` check.
- **Root cause:** The handler trusts message content, and either skips origin verification or implements it incorrectly, so any attacker-controlled window can feed the sink.
- **Exploitation steps:**
  1. Find the listener and trace `e.data` to its sink. Vulnerable example: `window.addEventListener('message', function(e){ eval(e.data); })`.
  2. Host an attacker page that frames the target and posts a payload after load: `<iframe src="//vulnerable-website" onload="this.contentWindow.postMessage('print()','*')">` (targetOrigin `*`).
  3. The listener accepts it (no origin check) and `eval` runs the payload. Swap `print()` for the real PoC code; for HTML sinks send markup, for `location`-style sinks send a `javascript:` URL (the labs cover the JavaScript URL and `JSON.parse` variants).
- **Bypasses used:** Flawed origin checks:
  - `if (e.origin.indexOf('normal-website.com') > -1)` is beaten by origin `http://www.normal-website.com.evil.net`.
  - `startsWith()` / `endsWith('normal-website.com')` are beaten by `http://www.malicious-websitenormal-website.com` (no dot boundary) or a longer attacker domain.
  - When the payload must be JSON, build the message so `JSON.parse(e.data)` yields the field the sink reads.
- **Impact:** Arbitrary JS in the victim page origin; reachable by luring the victim to an attacker page that frames or opens the target.
- **Bounty:** n/a
- **Lesson / pattern:** Treat every `message` handler as an untrusted-input source: require exact-match `e.origin === 'https://trusted.example'` (no substring or suffix tests) and never feed `e.data` to eval/innerHTML-like sinks.
- **Source:**
  - Title: Controlling the web message source (DOM XSS via postMessage)
  - Author: PortSwigger (Web Security Academy)
  - URL: https://portswigger.net/web-security/dom-based/controlling-the-web-message-source
  - Date: not stated (scraped 2026-11-16)
  - Disclosed report: none
  - Raw file: `sources/xss/18-portswigger-web-message-source.md`

Tags: dom, postmessage, filter-bypass, methodology
