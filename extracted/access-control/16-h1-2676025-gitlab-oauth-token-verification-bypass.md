# GitLab ROPC: the /oauth/token endpoint skips the login email-verification that web login enforces (program: bug, not vuln)

- **Vuln type / sub-variant:** authentication-control bypass via an alternate grant flow: the Resource Owner Password Credentials (ROPC) `/oauth/token` endpoint does not apply the new-IP/new-device email verification that web login does; with 2FA off, a leaked password yields a full-scope token from any IP
- **Target type:** GitLab authentication (gitlab.com `/oauth/token`, ROPC password grant)
- **Recon signal:** the app has an extra protection (email verification triggered on new IP or after failed logins) on one login path (web); check whether a second authentication path (ROPC token exchange) enforces the same control
- **Root cause:** the restriction on `/oauth/token` keys only on whether the account is active vs locked, not on device/IP or a per-request verification; the stricter email-verification safeguard lives only on the web login flow, so the alternate flow fails open on it
- **Exploitation steps:**
  1. Obtain the victim's username and password (precondition; 2FA disabled).
  2. From a new IP/VPS, POST `grant_type=password&username=...&password=...&scope=api` to `https://gitlab.com/oauth/token`.
  3. As long as the account is not locked, the server issues a full read/write access token and refresh token, with no email verification and no new-device notification that web login would trigger.
  4. Escalate to account takeover: add an attacker email via `POST /api/v4/user/emails` with the token, then use it to reset the password.
- **Bypasses used:** switching from web login to the ROPC grant on the same credentials to avoid triggering the IP/device email verification
- **Impact:** defeats the email-verification safeguard meant to protect 2FA-less accounts from remote/untrusted-device login; full-scope token and account takeover. Originally rated High (8.1).
- **Bounty:** none; GitLab closed it **Informative**, arguing the real root cause is the victim sharing a working password, so it is a bug not a vulnerability (ROPC is documented as high-trust only)
- **Lesson / pattern:** a security control is only as strong as the weakest authentication path; when an app adds verification/MFA on one flow, test every other grant type (ROPC, SSO, API tokens) for the same enforcement. Note this case shows disclosure disagreement: a bypass with a password precondition may be ruled a bug by the program.
- **Source:**
  - Title: Login email verification bypass via `/oauth/token`
  - Author: cybxis
  - URL: https://hackerone.com/reports/2676025
  - Date: 2024-09-03 (disclosed; reported 2024-08-22)
  - Disclosed report: https://hackerone.com/reports/2676025 (also https://cybxis.medium.com/a-bypass-on-gitlabs-login-email-verification-via-oauth-ropc-flow-e194242cad96)
  - Raw file: `sources/access-control/16-h1-2676025-gitlab-oauth-token-verification-bypass.md`

Tags: auth-bypass, ropc, oauth-token, email-verification-bypass, mfa-adjacent, alternate-grant-flow, informative-dispute, account-takeover
