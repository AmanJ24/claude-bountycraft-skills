# H1-2006 CTF chain: client-side 2FA hash, cookie-driven API path traversal, staff to admin via stored report replay

- **Vuln type / sub-variant:** multiple access-control and auth-bypass techniques chained in one CTF (BountyPay): (a) client-controlled 2FA challenge hash, (b) parameter/path injection in a base64 cookie to reach an IP-restricted host via an API redirect (SSRF), (c) a staff-creation API guarded only by a leaked static header token, (d) a UI-gated role upgrade abused by making the admin review bot replay attacker-influenced page state, (e) final payment 2FA bypass via the same SSRF. Only the access-control and auth-bypass checks are distilled here.
- **Target type:** fintech web app with a customer portal, staff portal, REST API, Android app, and an admin review bot
- **Recon signal:** a logger file referenced from a public GitHub repo exposed `/bp_web_trace.log`; a hidden form field `challenge`; a base64 cookie holding `account_id`; "401, not permitted from your IP" on an internal host; a page-report feature stating `/admin` pages are ignored; an `upgradeToAdmin` function in `website.js`
- **Root cause:** (1) the server trusts the 2FA expected-value MD5 hash supplied back by the client in a hidden input; (2) the cookie field `account_id` is concatenated into an internal API URL without validation, allowing `../` traversal into a redirect endpoint (SSRF inheriting internal network trust); (3) the staff-creation API relies only on a static `X-Token` and accepts any `staff_id` via POST; (4) the role-upgrade endpoint `/admin/upgrade?username=` is reachable only when an admin visits, so the admin bot rendering attacker-influenced page state performs the upgrade
- **Exploitation steps:**
  1. Fetch the exposed request log, base64-decode entries, recover valid login creds (`brian.oliver`) and a recorded 2FA answer.
  2. At the 2FA prompt, read the hidden `challenge` input (MD5 of the expected 10-char code); replace it with the MD5 of a known string (e.g. `1111111111`) and submit that string as the code. Login completes (client-side trusted 2FA).
  3. Decode the session cookie (`{"account_id":"F8gHiqSdpK","hash":...}`); set `account_id` to `../../redirect?url=https://software.bountypay.h1ctf.com/#`, re-encode as base64. The backend call becomes an SSRF through the API `redirect` endpoint and reaches the IP-restricted host that returned 401, used to enumerate directories and download `BountyPay.apk`.
  4. Decompile the APK (apktool), drive its intents via adb to recover a static `X-Token`; POST `staff_id=STF:8FJ3KFISL3` (a value leaked on Twitter) to `/api/staff`, which creates a staff account and returns its username and password with no authorization beyond the token.
  5. As staff, find `upgradeToAdmin` calling `/admin/upgrade?username=`; direct access returns "Only admins can perform this". Abuse the "Report This Page" admin bot: craft a staff URL with array `template[]` params and `#tab4` so the admin's render triggers the upgrade on the attacker's username, then reload Home to reveal the Admin tab (and the real admin creds marten.mickos).
  6. Log in as admin and complete the final payment, with the payment 2FA bypassed via the same SSRF path.
- **Bypasses used:** client-supplied MD5 hash as the 2FA oracle; base64 cookie tampering plus `../` path traversal into a redirect for SSRF; a leaked static `X-Token` as sole API auth; forcing the admin bot to execute a UI-gated `/admin/upgrade` through attacker-controlled page state (confused deputy)
- **Impact:** full compromise of the BountyPay application: unauthorized login, internal host access, staff account creation, staff-to-admin privilege escalation, and an unauthorized payment. CTF scenario (no CVSS); demonstrates OWASP A01 access-control failure classes end to end.
- **Bounty:** not stated (H1-2006 CTF writeup, not a paid bounty)
- **Lesson / pattern:** never let the client hold the answer to its own check (the 2FA hash), never build an internal URL from a client-controlled cookie field, never treat a static header token as authorization, and never gate a privileged action only in the UI where a server-side bot will faithfully replay attacker-influenced state. Each link is a server-side authorization that was delegated to the client.
- **Source:**
  - Title: [H1-2006 2020] Bypassing access control checks by modifying the URL, internal application state, or the HTML page, or using a custom API attack tool
  - Author: bcobain23
  - URL: https://hackerone.com/reports/895172
  - Date: 2020-06-10
  - Disclosed report: https://hackerone.com/reports/895172
  - Raw file: `sources/access-control/14-h1-895172-h12006-access-control-bypass-ctf.md`

Tags: ctf, access-control-chain, client-side-2fa, ssrf, path-traversal, static-token-auth, mflac, vertical-privesc, admin-bot-confused-deputy, force-browsing
