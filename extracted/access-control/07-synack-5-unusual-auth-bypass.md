# Synack: five unusual authentication bypasses, each a logic or config flaw scanners miss

- **Vuln type / sub-variant:** authentication bypass (research roundup of 5 real engagement findings): (1) token-refresh trusts client username, (2) SSO redirect leaks internal response, (3) CMS portlet direct access, (4) JWT audience not checked, (5) auth-type parameter set to accept empty password
- **Target type:** enterprise web apps using Bearer tokens, Microsoft SSO, Liferay CMS, JWT, and base64 XML login
- **Recon signal:** each flaw showed up only by watching how the app behaves under unexpected input, not from a signature: a `/refresh/tokenlogin` call carrying a username, an oversized (40k byte) redirect response, numeric portlet ids, auth-free JS files, a login `type` attribute on a hashed field
- **Root cause:** authentication decisions made client-side or left unvalidated server-side: username trusted on refresh, 302 response body fully populated before redirect, portlet routing not access-controlled, JWT accepted on signature alone without audience check, login parameter `type` controls how the password value is interpreted
- **Exploitation steps:**
  1. Refresh endpoint: drop the Authorization header and set `username` to any account; server mints a valid token for that user (anonymous to any-user).
  2. SSO redirect: intercept the 302, change it to 200 OK and delete the Location header; the leaked internal response renders the whole app.
  3. Liferay: change `p_p_id` to 58 to reach the login portlet, which exposes Create Account self-registration behind the single public login page.
  4. JWT: present a generic sample JWT (e.g. Microsoft's docs token) with a valid signature; backend accepts it because it never checks the token was issued for this app.
  5. XML login: set the `scode` `type` attribute to 0 and send an empty password value; server authenticates any username without checking credentials.
- **Bypasses used:** header deletion, response code/Location tampering (Burp Match and Replace), parameter id enumeration, reused sample token, parameter-type plus empty-value coercion
- **Impact:** token for any user, full unauthenticated app access, self-registration into internal apps, internal data access, and account takeover with an empty password
- **Bounty:** not stated (Synack platform findings)
- **Lesson / pattern:** trace what happens to a request that satisfies every check, not just the reject path; auth bypasses look like a client value trusted, a response built before a redirect, an identifier used as a credential, or an edge-case parameter value (null, empty, type=0) the server never anticipated.
- **Source:**
  - Title: Exploits Explained: 5 Unusual Authentication Bypass Techniques
  - Author: Synack Red Team researcher
  - URL: https://www.synack.com/exploits-explained/exploits-explained-5-unusual-authentication-bypass-techniques/
  - Date: unknown (page updated 2026-09-02)
  - Disclosed report: none
  - Raw file: `sources/access-control/07-synack-5-unusual-auth-bypass.md`

Tags: auth-bypass, jwt-audience, sso-misconfig, response-tampering, cms-portlet, empty-password, client-trusted-value, research
