# Mozilla Accounts: account/destroy acts on body email, not the session account

- **Vuln type / sub-variant:** Session-object misbinding. The endpoint authenticates the session but acts on the account named in the JSON body (email-keyed, not a numeric id).
- **Target type:** Account management REST API (`POST /v1/account/destroy`), including SSO-created accounts.
- **Recon signal:** The delete request carries the target account identity (`email`, plus an `authPW` credential field) in the body, separate from the session token. The hunter replayed the victim's destroy body under the attacker's session.
- **Root cause:** The server did not verify that the authenticated session belongs to the account being deleted. For password-based accounts the `authPW` check partly masked this, but for SSO-only accounts (no password ever set) the credential proof was effectively absent, so knowing the victim's email was enough.
- **Exploitation steps:**
  1. First variant: capture a victim's destroy body (email and password hash), then replay it from the attacker's session. Triage called this low value because it needs victim credentials.
  2. Stronger variant: create two SSO accounts that never set a password (attacker and victim).
  3. As the attacker, intercept your own destroy request, cancel it, and edit only the `email` field to the victim's address.
  4. Send it. The victim's account is permanently deleted.
- **Bypasses used:** none. The severity argument shifted from "needs credentials" to "email only" for SSO-only accounts.
- **Impact:** Permanent deletion of any SSO-only account knowing just its email. No takeover. Mozilla rated it high (initially closed informative, then low, then raised). Possible regression of an older bug per Mozilla staff.
- **Bounty:** Awarded, amount not stated in source.
- **Lesson / pattern:** For destructive endpoints, replay the request with another user's identifier in the body while keeping your own session. Also test the account type where the secondary proof (password) does not exist.
- **Source:**
  - Title: IDOR: Account Deletion via Session Misbinding - Attacker Can Delete Victim Account
  - Author: z3phyrus
  - URL: https://hackerone.com/reports/3154983
  - Date: reported 2025-05-20, disclosed 2025-06-03
  - Disclosed report: https://hackerone.com/reports/3154983
  - Raw file: `sources/idor/05-h1-3154983-mozilla-fxa-account-destroy-idor.md`

Tags: session-misbinding, delete, api, account-takeover
