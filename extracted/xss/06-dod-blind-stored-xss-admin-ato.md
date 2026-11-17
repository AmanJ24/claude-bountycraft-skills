# DoD: blind stored XSS via profile fields, fires in admin panel, admin account takeover

Source completeness: partial (DoD program, host redacted; terse body, no sink code, payload and impact described in prose only).

- **Vuln type / sub-variant:** Blind stored XSS with out-of-band exfiltration. Profile fields stored and rendered unescaped in an admin view; execution detected via an XSS Hunter callback.
- **Target type:** User profile page fields (first name, last name, company name, title) on a Department of Defense web application, rendered later in an admin panel.
- **Recon signal:** The profile-page fields did not strip HTML tags or JavaScript, so stored values could carry markup. Blind-XSS instinct: a profile reviewed by an admin is a likely second-order sink, so an OOB hook can surface it.
- **Root cause:** Profile field values are stored and written into the admin-panel HTML without sanitization or encoding. Unescaped sink: stored profile field to HTML in the admin view (an injected `<img src=... onerror/callback>` fetches the attacker host). Failed defenses: input sanitization and output encoding; admin session cookie exposed to the firing script.
- **Exploitation steps:**
  1. Log in and set the First name, last name, company name, and title profile fields to a blind-XSS payload that loads from an xss.ht hook.
  2. Wait for an administrator to view the profile.
  3. The payload fires in the admin's browser and XSS Hunter reports the event with screenshot, IP, cookies, and other data.
- **Bypasses used:** None documented; fields accepted raw HTML. Multiple fields seeded to maximize the chance of hitting the sink.
- **Impact:** Leak of the administrator's session cookies, IP, backend service details, and internal PII; stolen admin cookies enable admin-panel access and account takeover. Rated Critical by the team.
- **Bounty:** Not stated.
- **Lesson / pattern:** Unsanitized profile fields rendered in an admin console are a reliable blind-stored-XSS to admin-takeover path; seed several fields with an OOB hook and encode on output in every internal view.
- **Source:**
  - Title: Blind Stored XSS on [redacted] leads to takeover admin account
  - Author: hemantsolo
  - URL: https://hackerone.com/reports/1110243
  - Date: 2021-03-11 (disclosed)
  - Disclosed report: https://hackerone.com/reports/1110243
  - Raw file: `sources/xss/06-dod-blind-stored-xss-admin-ato.md`

Tags: stored, blind, admin-fire, session-theft, account-takeover
