# Grammarly/Superhuman: cookie-reflected XSS plus cookie-write endpoint, HttpOnly bypass, ATO

- **Vuln type / sub-variant:** Cookie-reflected XSS (HackerOne labels it Stored, since the payload persists in a cookie). Context: raw HTML, an unencoded copy of the cookie value inside a `<noscript>` block. Turned from self-XSS into a real attack by a cross-site cookie-write primitive on a sibling subdomain.
- **Target type:** Marketing/web-app page (www.grammarly.com/upgrade) plus a cookie-management API (gnar.grammarly.com/cookies).
- **Recon signal:** The hunter first noticed that gnar.grammarly.com/cookies lets any origin POST (set) or GET (read) any cookie scoped to `*.grammarly.com`, with credentials and no Referer check. That first report was dismissed as normal cookie behavior, so they hunted for a page that reflected a cookie they could now control. They found `gnar_containerId` echoed in two places on the upgrade page.
- **Root cause:** `gnar_containerId` is reflected into two `<img src>` values. One copy is encoded, the second copy inside `<noscript>` is not. Unescaped sink: cookie value to HTML inside `<noscript>`. Failed defenses: HttpOnly plus Secure on the `grauth` session cookie (defeated because the `/cookies` endpoint returns cookie values to a credentialed XHR sent from the Grammarly origin), no allowlist of settable cookies, no Referer/Origin check on the POST, and implicit trust in cookie content.
- **Exploitation steps:**
  1. Attacker page (served over HTTPS to avoid mixed-content blocking) sends a credentialed cross-site `POST https://gnar.grammarly.com/cookies?name=gnar_containerId&value=<payload>` to plant the stage-1 cookie.
  2. Stage-1 cookie value breaks out of the `<noscript>` block and loads an external script from the attacker host (the script tag is split in JS so the host page survives).
  3. The same page redirects the victim to the upgrade page, where the cookie is reflected into the `<noscript>` block and the external script loads inside the grammarly.com origin.
  4. The attacker script, running as grammarly.com, does a credentialed XHR GET to gnar.grammarly.com/cookies to read the HttpOnly `grauth` session cookie (same-origin, so the response is readable), then sends it to the attacker host.
- **Bypasses used:** HttpOnly/Secure bypass via the cookie-read endpoint; cross-site forced cookie-write (no CSRF/Referer protection) to deliver the payload; mixed-content avoided by HTTPS hosting; script-tag splitting so the surrounding page keeps rendering.
- **Impact:** Theft of the victim's session cookies (`grauth`, `csrf-token`), confirmed usable to fetch a victim's private documents; also allowed forced-session (login) by overwriting the victim's session cookie. Full account takeover.
- **Bounty:** Awarded by Superhuman, amount hidden. Severity High (8.2).
- **Lesson / pattern:** A cookie you can write cross-site plus any page that reflects that cookie unescaped equals real XSS, and an HttpOnly session cookie is not safe if a sibling endpoint hands its value back to a same-origin script. Encode cookie values on output and never let one subdomain read/write arbitrary cookies for the whole domain.
- **Source:**
  - Title: Account takeover through the combination of cookie manipulation and XSS
  - Author: k4r4koyun
  - URL: https://hackerone.com/reports/534450
  - Date: 2019-12-03 (disclosed)
  - Disclosed report: https://hackerone.com/reports/534450
  - Raw file: `sources/xss/01-grammarly-cookie-xss-ato.md`

Tags: stored, cookie-xss, session-theft, account-takeover, filter-bypass
