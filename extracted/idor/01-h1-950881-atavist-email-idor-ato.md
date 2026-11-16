# Atavist: body user id on email-change request gives zero-click account takeover

- **Vuln type / sub-variant:** Numeric/sequential ID IDOR in a write action (change email), chained with forgot-password to account takeover.
- **Target type:** Authenticated account settings form (reader account page on a CMS-style publishing platform), form POST.
- **Recon signal:** The hunter inspected the email-change request from his own settings page and saw a user `id` field in the request data. A client-supplied user identifier on a profile-write request is the classic cue.
- **Root cause:** The server trusted the `id` in the request body to pick which account to update. It never compared that id to the authenticated session's user. User ids were also sequential, so targets were enumerable.
- **Exploitation steps:**
  1. Log in with an attacker account and open the account settings page with a proxy on.
  2. Submit an email change and capture the request.
  3. Replace the `id` value with the victim's (test) account id and forward it, so the victim's email becomes an attacker-controlled address.
  4. Use the public forgot-password flow against that new email to reset the victim's password and log in.
- **Bypasses used:** none.
- **Impact:** Account takeover without any victim interaction. Because ids are sequential, every account could be taken over in bulk. The reporter also noted the endpoint accepted already-registered emails (minor side issue).
- **Bounty:** Awarded by Automattic, amount not stated in source.
- **Lesson / pattern:** Any write request carrying the object owner's id in the body (profile, email, phone) should be retested with another user's id, and "change email" plus "forgot password" is the standard escalation to takeover.
- **Source:**
  - Title: IDOR when editing email leads to Account Takeover on Atavist
  - Author: bugra
  - URL: https://hackerone.com/reports/950881
  - Date: reported 2020-08-04, disclosed 2020-11-18
  - Disclosed report: https://hackerone.com/reports/950881
  - Raw file: `sources/idor/01-h1-950881-atavist-email-idor-ato.md`

Tags: numeric-id, change-email, forgot-password, account-takeover, enumeration
