---
title: "How an OAuth 2.0 Redirect URI Bypass Led to Account Takeover and an $8,000 Bounty"
author: T4nv1 (@t4nv1)
url: https://medium.com/@t4nv1/how-an-oauth-2-0-redirect-uri-bypass-led-to-account-takeover-and-an-8-000-bounty-ad67ba4b0db6
date_published: 2026-08-27
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: $8,000
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- STATUS: PAYWALLED / PARTIAL. Medium member-only story. Only the public preview was saved; the actual bypass technique, exploitation and impact sections are behind the paywall and were NOT bypassed. -->

Member-only story. 3 min read. Aug 27, 2026. Tags: Cybersecurity, Security, Bug Bounty, Bug Bounty Tips, Hacking.

Single Sign-On (SSO) integrations are designed to streamline authentication, but subtle oversights in how callback parameters are validated can compromise the entire authentication flow. When an application relies on OAuth 2.0 without enforcing strict string matching on redirect URIs, attackers can intercept authorization codes and compromise user accounts.

While reviewing an enterprise analytics platform that allowed users to log in via a popular third-party identity provider, analyzing the authorization handshake revealed an **OAuth 2.0 Response Handling Flaw**, earning an **$8,000 bounty award**.

## Phase 1: Mapping the OAuth 2.0 Flow

When a user initiates authentication via SSO, the client browser is redirected to the identity provider's authorization endpoint:

HTTP

```
GET /oauth/v2/authorize?client_id=analytics_app_id&response_type=code&redirect_uri=https://analytics.platform.com/auth/callback&scope=openid%20profile%20email HTTP/1.1
Host: identity-provider.com
```

Upon successful user consent, the identity provider sends an authorization code back to the specified `redirect_uri`:

HTTP

```
HTTP/1.1 302 Found
Location: https://analytics.platform.com/auth/callback?code=AUTH_CODE_TEMP_12345
```

The backend server receives this temporary `code` and exchanges it server-to-server for an access token to authenticate the user.

## Phase 2: Testing Redirect URI Validation

To evaluate how strictly the identity provider and client application validated the `redirect_uri` parameter, several manipulation techniques were tested:

- **Parameter Alteration:** Changing the domain completely (`redirect_uri=https://attacker.com`). _Result: Rejected._
- **Subdomain Bypass:** Appending an arbitrary subdomain (`redirect_uri=https://attacker.analytics.platform.com`). _Result: Rejected._
- **Path Traversal / Query Injection:** Testing parameter manipulation inside the path structure.

## Create an account to read the full story.

The author made this story available to Medium members only.

[Paywall reached. Remainder of article not captured.]
