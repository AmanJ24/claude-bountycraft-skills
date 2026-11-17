# Exploiting postMessage handlers for DOM XSS (Synack)

Reference / technique writeup drawn from Synack Red Team engagements. Note: the post's PoC code listings were embedded as images and were not captured, so this note is reconstructed from the surviving prose.

- **Vuln type / sub-variant:** DOM-based XSS through an insecure `window.addEventListener('message', ...)` handler. Three origin-check failure modes: missing check, naive substring check, and null-origin acceptance.
- **Target type:** Pages that register a postMessage listener and route `message.data` into a DOM sink: an iframe/window redirect, a script `src`, or an `innerHTML` write. Often found on otherwise blank or uninteresting helper pages discovered via devtools.
- **Recon signal:** A page with a message event listener whose origin validation is absent or weak, where received data feeds a navigation, element creation, or HTML assignment. Open devtools and inspect registered listeners and any embedded iframe.
- **Root cause:** postMessage was meant for safe cross-origin messaging, but the receiver must validate `event.origin` strictly and must not treat attacker-controlled `message.data` as trusted. When origin validation is missing or flawed, any site can drive the handler, and the untrusted `data` reaches an execution sink.
- **Exploitation steps (data.data -> sink flow, by failure mode):**
  1. No origin check (Example 1): the handler redirects an iframe to a URL taken from the message. The same-origin policy blocks a direct `javascript:` navigation on a cross-origin iframe, so the attacker first messages a same-origin URL to make the iframe same-origin with the parent, then sends a second message carrying a `javascript:` URI, which now executes. Two-step redirect.
  2. Naive includes() check (Example 2): the handler guards with `event.origin.includes("targetsite.com")`. `includes()` returns true whenever the trusted string appears anywhere in the origin, so an attacker-registered host such as `https://targetsite.com.attacker.example` passes the check, and the handler then places attacker input into a script `src`, loading external JS.
  3. Null-origin bypass (Example 3): the handler compares `event.origin === window.origin` intending to allow same-origin messages. A sandboxed iframe sets both its MessageEvent origin and the `window.origin` of any window it opens to the literal string `"null"`, so both sides match and the message is accepted. The handler then writes the input to the DOM via `innerHTML`.
- **Bypasses used:** same-origin relay for `javascript:` URI on an iframe, substring/`includes()` origin confusion via crafted subdomain, sandboxed-iframe null-origin forcing.
- **Impact:** DOM XSS in the context of the vulnerable page (script execution, data/session access on that origin).
- **Bounty:** n/a (engagement examples, amounts not disclosed).
- **Lesson / pattern:** Validate `event.origin` with strict equality against an allowlist of exact trusted origins; never use `includes()`/substring matching, and compare against `window.location.origin` (not `window.origin`, which sandboxing can force to `"null"`). Treat `message.data` as untrusted and keep it out of navigation, `script.src`, and `innerHTML`.
- **Source:**
  - Title: Exploiting PostMessage Handlers to Achieve DOM XSS
  - Author: John Kounelis (Synack Red Team)
  - URL: https://www.synack.com/exploits-explained/exploiting-postmessage-handlers-dom-xss/
  - Date: 2026 (page modified 2026-08-20; exact publish date not shown)
  - Disclosed report: none
  - Raw file: `sources/xss/25-synack-postmessage-dom-xss.md`
  - Note: PoC code was images, not captured; mechanism reconstructed from prose.

Tags: postmessage, filter-bypass, methodology
