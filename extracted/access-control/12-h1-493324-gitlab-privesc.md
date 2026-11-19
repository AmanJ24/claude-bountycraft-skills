# GitLab: impersonated admin session exposed to the impersonated user

- **Vuln type / sub-variant:** vertical privilege escalation via session exposure (admin impersonation session leak)
- **Target type:** GitLab web app, admin impersonation feature and Active Sessions page
- **Recon signal:** admin impersonation creates a separate session; the target user's Active Sessions list may reveal it
- **Root cause:** the admin's session shows up in the victim's session list and its session id is exposed in the Revoke control; "Stop impersonating" then restores the admin session for whoever holds that cookie
- **Exploitation steps:**
  1. As attacker, revoke other sessions so the list is clean.
  2. Have an admin impersonate the attacker (needs a pretext such as a support request).
  3. Reload Active Sessions; a new session appears. Read its session id from the Revoke button markup.
  4. Visit the index page, clear cookies, and set `_gitlab_session` to the copied value.
  5. Refresh to be inside the impersonated session, then click "Stop impersonating" to land as the admin.
- **Bypasses used:** session cookie replay of the leaked id; requires admin interaction (UI: required)
- **Impact:** any authenticated user, even external, becomes instance admin; Critical (9.0)
- **Bounty:** awarded, amount hidden; fixed in GitLab 11.8.1
- **Lesson / pattern:** look for privileged-session artifacts (impersonation, sudo, support login) reachable from low-priv views; session identifiers must never be displayed to users.
- **Source:**
  - Title: Privilege escalation from any user (including external) to gitlab admin when admin impersonates you
  - Author: skavans
  - URL: https://hackerone.com/reports/493324
  - Date: 2019-02-09 (reported; disclosed 2020-08-26)
  - Disclosed report: https://hackerone.com/reports/493324
  - Raw file: `sources/access-control/12-h1-493324-gitlab-privesc.md`

Tags: vertical-privesc, impersonation, session-leak, session-id-exposed, cookie-replay, admin-takeover, ui-required
