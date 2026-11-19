# Auth0 cross-tenant authentication bypass: one tenant's /authorize signs a valid JWT for a user who exists only in another tenant

- **Vuln type / sub-variant:** cross-tenant authentication bypass / identity impersonation; the `/authorize` endpoint fails to validate which tenant and client a session (`sid`) belongs to before issuing tokens, so a session from an attacker tenant yields a correctly signed JWT ID token from a victim tenant
- **Target type:** Auth0 (Okta Customer Identity Cloud) multi-tenant IDaaS; public SPAs with open `/authorize` endpoints
- **Recon signal:** apps that make authorization decisions by trusting ID-token claims (especially `email`), combined with a multi-tenant IdP where tenant domains and client IDs are public; relevant whenever a backend trusts a JWT's email/identity claim
- **Root cause:** if a session has any tenant information in it, `/authorize` does not check that the session is valid for the requesting tenant and client before signing tokens; the victim tenant signs a JWT, with its own correct key, for an email that only exists in the attacker tenant
- **Exploitation steps:**
  1. Learn the victim tenant domain, a client ID in it (public for SPAs), and the victim's email; have your own tenant in the same locality.
  2. In the attacker tenant, create a user with the victim's email address (and set email as verified; optionally disable the verification email so the victim is not warned).
  3. Authenticate as that user in the attacker tenant (Universal Login or `/co/authenticate`); obtain the Auth0 session cookie.
  4. Replay the `/authorize` request but swap the domain and client ID to the victim tenant's values, sending the attacker session cookie.
  5. The victim tenant signs and returns a JWT ID token for the victim's email, though no such user exists there.
- **Bypasses used:** cross-tenant session reuse; self-asserting `email_verified` in your own tenant; note the forged token keeps the attacker's `sub`, so apps checking `sub` resist it, but apps trusting `email` do not
- **Impact:** impersonate any victim in an Auth0 customer's system knowing only their email (when the app trusts the email claim); conceptually the same class as Descope's nOAuth. Auth0 silently changed the behaviour; not formally confirmed.
- **Bounty:** not stated (coordinated disclosure; reporter's employer bars bug bounty participation)
- **Lesson / pattern:** always validate which tenant and client a session/token is valid for before trusting any claim in it, and never make authorization decisions on a mutable, attacker-settable claim like `email`; prefer `sub` and verify issuer plus audience. ID tokens are for UX, not authorization.
- **Source:**
  - Title: Vulnerability disclosure: Authentication bypass in Auth0
  - Author: Laban Skollermark (Sentor / Accenture Security)
  - URL: https://sentorsecurity.com/blog/vulnerability-disclosure-authentication-bypass-in-auth0/
  - Date: 2023-10-24
  - Disclosed report: none (vendor acknowledged receipt only)
  - Raw file: `sources/access-control/17-sentor-auth0-tenant-claim-bypass.md`

Tags: auth-bypass, cross-tenant, multi-tenant-isolation, oauth, oidc, jwt-claim-trust, email-claim, noauth-class, impersonation, idp
