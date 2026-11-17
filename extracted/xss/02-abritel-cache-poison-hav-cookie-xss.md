# Abritel (Expedia): cache-poisoned stored XSS via reflected cookie, HttpOnly bypass, ATO

- **Vuln type / sub-variant:** Cache-poisoned stored XSS. A cookie value reflected into a JS-string context (`var hav="..."`), broken out into HTML, then persisted for all visitors via web-cache poisoning. Cookie-reflected plus cache-poisoning hybrid.
- **Target type:** Cacheable page on a travel site (a `.php.js` listing endpoint on www.abritel.fr).
- **Recon signal:** The `hav` cookie was reflected into the page body as a JavaScript variable assignment (`var hav="value from cookie"`). The page was served with a cacheable extension/path, suggesting the reflected value could be baked into the shared cache and served to other users.
- **Root cause:** The `hav` cookie is reflected unescaped into a double-quoted JS string, which is also inside a `<script>` block in the HTML. Unescaped sink: cookie to JS-string/HTML. The page is treated as cacheable, so the poisoned response is stored and served to every later visitor. Failed defenses: a WAF (bypassed, see below), output encoding (absent), and HttpOnly on the session cookie (defeated because the session was also present in a readable JS variable).
- **Exploitation steps:**
  1. Send a request to the cacheable listing URL with a crafted `hav` cookie whose value closes the `</script>` and injects an HTML element with an event handler.
  2. The server reflects the value and returns a response the cache stores as the canonical page.
  3. Any other user who visits the same URL in a normal browser receives the poisoned cached page and the script executes in their session.
  4. Weaponized variant reads the session value exposed in the `window.INITIAL_STATE.system.cookie` JS variable and exfiltrates it.
- **Bypasses used:** WAF bypass by abusing the server's own behavior of stripping double quotes: the hunter inserted stray double quotes inside tag/keyword names (for example `</sc"ript>` and `sv"g`), which the WAF did not recognize as an attack, but the server removed the quotes and produced valid `</script><svg ...>` on output. The WAF filtered `<`/`>` tokens less strictly than the quote handling, so angle brackets passed.
- **Impact:** Stored XSS served to arbitrary visitors; theft of the HttpOnly session (`HASESSIONV3`) read from the in-page JS state variable, leading to account takeover.
- **Bounty:** Awarded, amount hidden. Severity High (8.2). Fixed by no longer reflecting the `hav` cookie.
- **Lesson / pattern:** A reflected cookie on a cacheable page becomes stored XSS for everyone via cache poisoning, and a WAF that mutates input (stripping quotes) can be turned into the bypass itself. HttpOnly is moot when the session is also dumped into a client-side state object.
- **Source:**
  - Title: Cache Poisoning Allows Stored XSS Via hav Cookie Parameter (To Account Takeover)
  - Author: bombon
  - URL: https://hackerone.com/reports/1760213
  - Date: 2023-04-01 (disclosed; submitted 2022-11-02)
  - Disclosed report: https://hackerone.com/reports/1760213
  - Raw file: `sources/xss/02-abritel-cache-poison-hav-cookie-xss.md`

Tags: stored, cache-poisoning, cookie-xss, waf-bypass, session-theft, account-takeover
