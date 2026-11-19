# Inflection: read-only user edits own privileges via API PUT (source is partial)

- **Vuln type / sub-variant:** vertical privilege escalation, missing function-level access control (UI hides, API does not enforce)
- **Target type:** web app with a REST API and role-based users page
- **Recon signal:** the users-management page is hidden from read-only users in the UI, implying the backend may still accept the underlying API calls
- **Root cause:** role restrictions applied only in the front end; the API endpoint for updating users does not check that the caller may change privileges
- **Exploitation steps:** (source is partial; only the program summary is public)
  1. Log in as a read-only user.
  2. Identify the API call used by the (hidden) users page to update a user.
  3. Send a PUT to that API changing the account's own privileges.
  4. Result: account holds elevated rights up to admin.
- **Bypasses used:** none beyond calling the API directly; exact endpoint and body are not public
- **Impact:** read-only account becomes admin; rated High (8.2) (source is partial)
- **Bounty:** awarded, amount hidden
- **Lesson / pattern:** a hidden UI page is not authorization; replay every admin-capable API call from low-privilege sessions, especially self-targeted updates of role/privilege fields.
- **Source:**
  - Title: Privilege Escalation: Read-Only to Admin
  - Author: foobar7
  - URL: https://hackerone.com/reports/277138
  - Date: 2019-03-15 (reported 2017-10-14)
  - Disclosed report: https://hackerone.com/reports/277138
  - Raw file: `sources/access-control/08-h1-277138-readonly-to-admin-mflac.md`

Tags: vertical-privesc, mflac, front-end-only, client-side-enforcement, privilege-escalation, admin-access, missing-server-check
