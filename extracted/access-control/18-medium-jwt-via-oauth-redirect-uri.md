# JWT theft via unvalidated OAuth redirect_uri (implicit flow leaks id_token to attacker host)

- **Vuln type / sub-variant:** OAuth redirect_uri not validated, combined with implicit flow (`response_type=id_token`); authorization response (the JWT) delivered to an attacker-controlled domain, leading to account takeover
- **Target type:** web app with a chatbot "Support Ticket" button that starts an OAuth flow between `app.domain.com` and `support.domain.com` (Freshworks JWT customer SSO)
- **Recon signal:** clicking the support link fires an OAuth redirect carrying `redirect_uri=https://support.domain.com/callback`; the `redirect_uri` parameter is attacker-reachable and worth testing for open-redirect behaviour
- **Root cause:** the authorization endpoint accepts any `redirect_uri` with no allowlist or origin check, and the flow returns the id_token in the URL (implicit flow); a token is therefore sent wherever the attacker names
- **Exploitation steps:**
  1. Intercept the OAuth request: `GET /account/freshworks/jwt/customer?response_type=id_token&client_id=...&scope=openid email profile&state=...&redirect_uri=https://support.domain.com/callback`.
  2. Change `redirect_uri` to `https://google.com`; the app shows "Login Successful" and offers to redirect there, confirming no validation.
  3. Set `redirect_uri` to a Burp Collaborator host; forward the request.
  4. Collaborator receives `GET /?id_token=<JWT>&client_id=...&state=...`, the victim's full JWT leaked as a URL parameter.
  5. Deliver the crafted link to an already-authenticated victim (phishing as a support email); their token is captured on click.
- **Bypasses used:** redirect_uri swap to an external domain; Burp Collaborator to prove exfiltration; relies on the victim already having a session on app.domain.com
- **Impact:** complete account takeover of any authenticated user who clicks the link; mass token harvesting possible
- **Bounty:** not stated (responsibly disclosed and patched)
- **Lesson / pattern:** strictly allowlist redirect_uri server-side and prefer Authorization Code flow with PKCE over implicit; a token returned in a URL to an unvalidated destination is account takeover, not a cosmetic open redirect. Always test OAuth in an authenticated session state.
- **Source:**
  - Title: Stealing JWT Tokens via OAuth redirect_uri Manipulation
  - Author: Shah kaif (@SKaif009)
  - URL: https://infosecwriteups.com/stealing-jwt-tokens-via-oauth-redirect-uri-manipulation-a-critical-vulnerability-abbd579b5443
  - Date: 2025-10-03
  - Disclosed report: none
  - Raw file: `sources/access-control/18-medium-jwt-via-oauth-redirect-uri.md`

Tags: oauth, redirect-uri, implicit-flow, token-leak, jwt, account-takeover, open-redirect, collaborator
