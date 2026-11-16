# US DoD web portal: ID parameter on profile update swaps victim email for takeover

Source completeness: partial (target domain, the request itself and the video are redacted by the program; a longer blog post exists on the reporter's site but is not in this raw file)

- **Vuln type / sub-variant:** Numeric/sequential-style ID IDOR (exact id format not shown) on a profile-update POST, chained with forgot-password to takeover.
- **Target type:** Authenticated account page of a portal reached after a sign-in on a military-related domain.
- **Recon signal:** The hunter registered an account, clicked "update" on the account page and captured the request. The reporter says he assumed id=1 would be the admin account, so the request carries a user id. Anything beyond that is not stated in source.
- **Root cause:** Two missing controls: the server did not tie the id in the request to the session user (no 403/401 for acting on another account), and the email change did not require the current password.
- **Exploitation steps:**
  1. Create an attacker account and log in.
  2. Open the account page, press update, and capture the request.
  3. Change the id parameter to the victim's and set the email to an attacker address.
  4. Trigger forgot password to receive the reset link and take over the account.
- **Bypasses used:** none stated.
- **Impact:** Account takeover. Severity was later raised to critical by the program. The sign-in page redirect made the host in scope per the reporter.
- **Bounty:** None (VDP).
- **Lesson / pattern:** Email change without re-authentication amplifies any ownership flaw into takeover. Suggested fixes: require old password, return 403 on cross-account writes.
- **Source:**
  - Title: IDOR to Account Takeover on https://[redacted]/index.html
  - Author: nagli
  - URL: https://hackerone.com/reports/969223
  - Date: reported 2020-08-27, disclosed 2020-09-29
  - Disclosed report: https://hackerone.com/reports/969223
  - Raw file: `sources/idor/04-h1-969223-dod-idor-ato.md`

Tags: numeric-id, change-email, forgot-password, account-takeover
