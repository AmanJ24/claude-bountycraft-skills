# 0xn3va OAuth 2.0 cheat sheet: client-side and authorization-server flaws, split by where the bug lives

- **Vuln type / sub-variant:** reference cheat sheet of OAuth 2.0 vulnerabilities organized by component: client-application issues (implicit-grant trust, state/CSRF, secret disclosure, open redirect) and authorization-server issues (redirect_uri weaknesses, host-header poisoning, scope upgrade, email-based account assignment, acr/amr 2FA bypass, session poisoning)
- **Target type:** apps using OAuth login and the authorization servers behind them; especially implicit-grant SPAs and apps offering both password and social login
- **Recon signal:** an implicit-grant app that POSTs `{user_id, access_token}` to its own server to create a session; a `state` that is missing/static/unvalidated; a `redirect_uri` that can be nudged off-domain; login that also allows social sign-in for the same email
- **Root cause:** the server implicitly trusts client-supplied data: it does not confirm the `access_token` matches the other identity fields, does not enforce `state`, does not strictly validate `redirect_uri`, derives the authorized client from session rather than the request, or trusts an unverified email to merge accounts
- **Exploitation steps (key vectors):**
  1. Implicit-grant impersonation: when the app submits `user_id` + `access_token` to its server to log in, change `user_id` to the victim; if the server does not check the token belongs to that user, you are logged in as them.
  2. state/CSRF: with a missing/static/unvalidated `state`, pause your own flow after authorization and send the URL to a logged-in victim to graft your account onto theirs.
  3. Email-based assignment: if the app (or the IdP) does not verify email on signup, pre-create an account with the victim's email; when they later use social login, the app links it to your account.
  4. Host-header poisoning: poison `Host` so the authorization `code`/token is exchanged against your domain (as in the Periscope ATO).
  5. acr/amr downgrade: if the server honours `amr_values`/`acr_values`, request `amr_values=pwd` instead of `pwd+otp` to bypass 2FA.
  6. redirect_uri session poisoning: when the authorize POST omits client params, poison the session with an untrusted `client_id` via a hidden cross-domain request so the victim's approval redirects to the attacker client's `redirect_uri`.
  7. Scope upgrade: as the client, add extra `scope` to the code-exchange request; if not validated against the original authorization, a higher-privileged token is issued.
- **Bypasses used:** weak redirect_uri configs (open redirect, path traversal, weak regex, parser discrepancies, HPP, `localhost.attacker.com`, Referer leak via HTML injection), host-header injection, amr downgrade, session poisoning, scope tampering on re-release
- **Impact:** account takeover via token/code theft or account linking, 2FA bypass, and privilege/scope escalation
- **Bounty:** not stated (reference; cites disclosed reports: Snapchat SSO token theft #265943, Uber FB OAuth token leak #202781, Periscope #317476)
- **Lesson / pattern:** never let the server implicitly trust client-submitted identity in implicit flows, enforce a per-session random `state`, strictly exact-match `redirect_uri`, bind the authorized client to the request not the session, verify email ownership before merging accounts, and do not let `acr`/`amr` parameters weaken the required factors.
- **Source:**
  - Title: OAuth 2.0 Vulnerabilities (Application Security Cheat Sheet)
  - Author: 0xn3va
  - URL: https://0xn3va.gitbook.io/cheat-sheets/web-application/oauth-2.0-vulnerabilities
  - Date: unknown (footer: updated ~3 years ago)
  - Disclosed report: none (reference aggregates several)
  - Raw file: `sources/access-control/05-0xn3va-oauth-cheatsheet.md`

Tags: oauth, implicit-grant, state-csrf, redirect-uri, host-header-poisoning, scope-upgrade, email-account-assignment, acr-amr-2fa-bypass, session-poisoning, account-takeover, taxonomy
