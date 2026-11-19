# DoD VDP: registration form user_type tampering yields admin

- **Vuln type / sub-variant:** vertical privilege escalation at registration via client-supplied role parameter (mass-assignment style)
- **Target type:** ColdFusion applicant-registration web app (DoD)
- **Recon signal:** the sign-up request carries a `user_type` field whose default value is 5, an obvious role selector sent by the client
- **Root cause:** the server trusts the submitted `user_type` when creating the account instead of fixing the role server-side. Odd input in name fields (`<%` sequences) was suspected to play a part.
- **Exploitation steps:**
  1. Proxy the registration flow and reach the new-user form.
  2. Fill it with a plausible, unused SSN.
  3. Intercept the submit and set `user_type=4`, with `fname` and `lname` containing `<%` fragments.
  4. Complete registration; accept the privacy page.
  5. The new account has admin access to all applicants' PII (SSNs, names, phones, emails). Reporter stopped there.
- **Bypasses used:** role parameter tampering; `<%` in name fields possibly needed; permissions looked delayed (an account with `user_type=1` later behaved as super admin)
- **Impact:** admin takeover and mass PII exposure; Critical
- **Bounty:** hidden (VDP)
- **Lesson / pattern:** diff signup requests for role/type/level fields and try other values; recheck accounts later, since privilege application may lag. Keep PoC data minimal and sanitized.
- **Source:**
  - Title: [Critical] Insufficient Access Control On Registration Page of Webapps Website Allows Privilege Escalation to Administrator
  - Author: hunt4p1zza
  - URL: https://hackerone.com/reports/796379
  - Date: 2020-02-14 (disclosed 2020-05-27)
  - Disclosed report: https://hackerone.com/reports/796379
  - Raw file: `sources/access-control/11-h1-796379-registration-privesc-admin.md`

Tags: vertical-privesc, parameter-role, mass-assignment-role, trust-user-input, admin-access, data-exposure, privilege-escalation
