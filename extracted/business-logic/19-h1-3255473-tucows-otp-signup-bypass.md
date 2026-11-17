# Signup OTP check skipped by omitting the code parameter (hover.com)

- **Vuln type / sub-variant:** OTP bypass via missing-parameter handling during account registration
- **Target type:** Domain registrar signup endpoint (JSON POST /signup)
- **Recon signal:** The signup request carried a `code` field holding the emailed OTP, suggesting the server might only validate it when present.
- **Root cause:** Server-side verification was conditional on the `code` key existing. When the key was absent, the OTP branch was never entered and registration completed anyway.
- **Exploitation steps:**
  1. Capture the normal signup request, which includes the OTP in the account object.
  2. Replay it with the `code` key deleted entirely (not just emptied).
  3. Receive a success response and a valid session; the account exists without proving ownership of the email.
- **Bypasses used:** Parameter removal (delete the key, not only the value) to reach an unvalidated code path.
- **Impact:** Register accounts with emails the attacker does not own, block real owners from registering, spam or impersonation. Program later reassigned the asset and ended with no severity.
- **Bounty:** None (VDP). Severity moved from medium 6.5 to 5.3, then none after an asset reassignment.
- **Lesson / pattern:** For every verification field (OTP, token, captcha), try deleting it, nulling it, and sending it empty. Treat "mandatory" as a client-side notion until the server proves otherwise.
- **Source:**
  - Title: Business Logic Error - Bypass of OTP Verification During Signup on hover.com
  - Author: c0rvuz
  - URL: https://hackerone.com/reports/3255473
  - Date: 2025-07-16
  - Disclosed report: https://hackerone.com/reports/3255473
  - Raw file: `sources/business-logic/19-h1-3255473-tucows-otp-signup-bypass.md`

Tags: otp-bypass, missing-server-validation, parameter-tampering, workflow-bypass, assumption-flaw
