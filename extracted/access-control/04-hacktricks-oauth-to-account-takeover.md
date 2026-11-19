# HackTricks OAuth-to-ATO: the full attack surface of the authorization-code grant, from redirect_uri to mutable claims

- **Vuln type / sub-variant:** reference taxonomy of OAuth 2.0 / OIDC account-takeover vectors across the authorization-code (and implicit) grant: redirect_uri weaknesses, state/CSRF, PKCE downgrade, provider mix-up, code replay/binding, token leakage, mutable-claim and client-confusion attacks
- **Target type:** apps using OAuth/OIDC login (SPAs, mobile with custom schemes, multi-IdP clients), authorization servers, and OAuth-capable desktop/agent clients
- **Recon signal:** any `/authorize` flow with an attacker-reachable `redirect_uri`, `state`, `scope`, or `response_mode`; callback/error pages that reflect `error_description` or opaque `state`; discovery docs at `/.well-known/openid-configuration` and `/.well-known/oauth-authorization-server`; SPAs that gate only client-side and trust `localStorage` identity
- **Root cause:** authorization decisions that trust attacker-influenced inputs: unvalidated redirect targets, missing per-browser `state`, codes/tokens not bound to issuer/client/redirect, mutable `email` used as identity instead of `iss`+`sub`, and backends that never verify the bearer token server-side
- **Exploitation steps (representative vectors):**
  1. redirect_uri: send a victim an authorize URL with `redirect_uri=attacker` (no validation, weak host regex, path traversal `/oauth/../`, wildcard subdomain takeover, IDN homograph, allowlisted-domain open redirect/XSS) to capture the victim's `code`/token.
  2. state/CSRF: if `state` is missing, static, optional, or unvalidated, complete your own flow, keep the final `?code=` request, and force the victim's browser to replay it so their account links to your IdP identity.
  3. PKCE downgrade: strip `code_challenge`, get an unbound code, inject it into the victim's pending callback when the token endpoint ignores the verifier.
  4. Mix-up: when `state`/`nonce`/PKCE live in provider-agnostic cookies, a value minted for provider A satisfies provider B's callback; replay across providers.
  5. Code handling: test lifetime, sequential reuse, concurrent redemption (race), and redeeming app A's code at app B's token endpoint (audience not bound).
  6. Mutable claims / client confusion: register your own org (e.g. Entra) and present an unverified `email=victim@...` where the RP trusts email; or harvest implicit-flow tokens on your site and replay them where the token's client_id is not checked.
  7. Token leakage: tokens in URL/history/Referer, localStorage, or a telemetry postMessage confused deputy; a username-only `ssoAuthentication` endpoint that signs a session for any supplied identity.
- **Bypasses used:** redirect allowlist evasion (regex, traversal, wildcard, homograph), state removal/fixation, PKCE strip, cross-provider cookie reuse, scope-upgrade on the code exchange, response_mode (query/fragment/form_post/web_message), `prompt=none`, ROPC for 2FA bypass, clickjacking the consent dialog, dynamic client registration of an attacker callback
- **Impact:** full account takeover on the victim app, token/scope theft, cross-tenant impersonation, and in edge cases SSRF (registration `logo_uri`/`jwks_uri`) or RCE (malicious discovery `authorization_endpoint` opened by desktop clients, CVE-2025-6514)
- **Bounty:** not stated (reference; cites many disclosed ATOs: Flickr/Cognito, Salt "Oh-Auth", Periscope host-header, AT&T SSO bypass)
- **Lesson / pattern:** bind every pending OAuth transaction to provider ID, exact issuer, client ID, redirect_uri, `state`, `nonce`, and PKCE verifier, consume it once, exact-match redirect_uri, keep tokens out of the browser/URL, and derive identity from verified `iss`+`sub` with `email_verified`, never a caller-supplied username or mutable email. Access tokens never belong in the resource owner's browser.
- **Source:**
  - Title: OAuth to Account Takeover
  - Author: HackTricks (carlospolop and contributors)
  - URL: https://hacktricks.wiki/en/pentesting-web/oauth-to-account-takeover.html
  - Date: unknown (living wiki)
  - Disclosed report: none (reference aggregates many)
  - Raw file: `sources/access-control/04-hacktricks-oauth-to-account-takeover.md`

Tags: oauth, oidc, redirect-uri, state-csrf, pkce-downgrade, provider-mixup, code-binding, mutable-claims, client-confusion, token-leak, dynamic-client-registration, account-takeover, taxonomy
