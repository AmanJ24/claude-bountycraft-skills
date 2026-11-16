# Pressable: application[id] on API app update moves apps, then leaks client secrets

- **Vuln type / sub-variant:** Numeric/sequential ID IDOR in a nested form parameter, escalated from write (reassign object) to read (error page rendering victim object) to credential theft.
- **Target type:** Web dashboard for managing API applications (OAuth-style client id and secret) on a managed hosting service.
- **Recon signal:** On the app update request the hunter noticed an `application[id]` body parameter. He tried his second account's app id and the app moved into his account, proving no ownership check.
- **Root cause:** The update action looked up the application purely by the supplied `application[id]` without scoping to the current user's applications. Even when validation failed (missing name), the error view rendered the looked-up object, including its client id and secret.
- **Exploitation steps:**
  1. Create an API application and trigger Update with a proxy on.
  2. Change `application[id]` to another account's app id. Observation: the app is reassigned (low impact on its own).
  3. Strip all other parameters, leaving only `application[id]` and the CSRF `authenticity_token`.
  4. The server returns a "name must be provided" error but renders the victim app page with Client ID and Client Secret.
  5. Use those credentials against the hosting provider's API (for example to add collaborators).
- **Bypasses used:** Removing the other parameters to force the validation-error path that leaks data instead of performing the write.
- **Impact:** Theft of any customer's API credentials, leading to account takeover via the public API. Ids are sequential so all applications were enumerable. The weaker write variant removes apps from victims.
- **Bounty:** Awarded by Automattic, amount not stated (reporter references two IDOR issues being rewarded).
- **Lesson / pattern:** When a write IDOR looks low impact, try minimal or invalid bodies: error paths often render the foreign object in full.
- **Source:**
  - Title: IDOR in API applications (able to see any API token, leads to account takeover)
  - Author: bugra
  - URL: https://hackerone.com/reports/1695454
  - Date: reported 2022-09-08, disclosed 2022-11-01
  - Disclosed report: https://hackerone.com/reports/1695454
  - Raw file: `sources/idor/06-h1-1695454-pressable-api-apps-idor.md`

Tags: numeric-id, api, account-takeover, mass-assignment, enumeration, pii-disclosure
