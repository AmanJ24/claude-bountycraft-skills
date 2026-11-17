# Automattic/Jetpack: DOM XSS via postMessage, origin check defeated by XSS on the allowed origin

- **Vuln type / sub-variant:** DOM-based XSS via web messaging (postMessage), chained. Two flaws combined: (1) a reflected/DOM XSS on the allowed-origin page (widgets.wp.com) and (2) a Jetpack `message` listener whose `event.origin` check trusts widgets.wp.com but passes a message field unencoded to `innerHTML`. The origin allowlist is defeated by owning XSS on the very origin it trusts.
- **Target type:** A sharing-buttons preview widget (widgets.wp.com/sharing-buttons-preview/) and the Jetpack Likes `postMessage` listener embedded on every WordPress/Jetpack site with Likes enabled.
- **Recon signal:** Jetpack's Likes feature registers a cross-frame `postMessage` listener that only accepts messages from https://widgets.wp.com. The hunter realized that an origin check is worthless if you can execute script on the allowed origin, then found a DOM XSS on that exact widgets.wp.com page via unescaped URL parameters.
- **Root cause:** Two unescaped sinks. (1) On widgets.wp.com, `custom[0][name]` and `custom[0][icon]` URL params are templated into the DOM unescaped, giving XSS on that origin. (2) In the Jetpack listener's `showOtherGravatars` path, the `liker.avatar_URL` field from the received postMessage is interpolated directly into an `element.innerHTML` template string. Message field: `avatar_URL`. Sink: `innerHTML`. Failed defenses: the `event.origin === 'https://widgets.wp.com'` check (bypassed by having code run on widgets.wp.com), and missing output encoding (an older plugin version applied `encodeURI` to avatar_URL but it was removed).
- **Exploitation steps:**
  1. Load the widgets.wp.com sharing-buttons-preview page with crafted `custom[0][name]`/`custom[0][icon]` params to get script execution on the widgets.wp.com origin.
  2. From that origin, send a `postMessage` to the Jetpack Likes frame/listener with a `liker.avatar_URL` value containing an HTML breakout. The origin check passes because the message genuinely comes from widgets.wp.com.
  3. The listener assigns the tampered `avatar_URL` into `innerHTML` on the host page (for example wordpress.com), executing script in that site's origin.
- **Bypasses used:** Origin allowlist bypass by pivoting through XSS on the trusted origin; the `avatar_URL` breakout out of an `<img src>` interpolation into an executable element.
- **Impact:** DOM XSS firing on any site running Jetpack Likes (the hunter estimated 100k+ domains), including wordpress.com and jetpack.com. Generic full client-side compromise of each affected user (perform/read/modify anything the user can). Rated High specifically because the reach is one XSS on widgets.wp.com cascading to every Likes-enabled site.
- **Bounty:** Awarded, amount hidden. Severity High.
- **Lesson / pattern:** An `event.origin` allowlist is only as strong as the most XSS-prone page on the allowed origin; a single XSS on a trusted widget origin turns every consumer's postMessage listener into XSS. Validate origin AND sanitize/encode every message field before any HTML sink.
- **Source:**
  - Title: DOM XSS on multiple Automattic domains through postMessages
  - Author: renniepak
  - URL: https://hackerone.com/reports/2371019
  - Date: 2024-02-26 (disclosed)
  - Disclosed report: https://hackerone.com/reports/2371019
  - Raw file: `sources/xss/08-automattic-dom-xss-postmessage.md`

Tags: dom, postmessage, filter-bypass, account-takeover
