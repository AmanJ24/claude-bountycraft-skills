# Unguessable lead IDs leaked by signup error, then used for PII read and phone update

- **Vuln type / sub-variant:** UUID/opaque ID IDOR (write, with PII exposure) where the "unguessable" ID is leaked through an enumeration oracle on the signup endpoint.
- **Target type:** Lead/customer API on a private program: `PUT /api/lead/<id>/phone-number` for updates and `POST /api/leads` for registration.
- **Recon signal:** The phone-number update call used a non-numeric lead `id` in the path. Testing with two accounts (container-isolated browser tabs) showed the server did not tie that ID to the session. Remaining problem was obtaining other users' IDs; known "unpredictable ID" tricks did not work until the signup endpoint was probed.
- **Root cause:** The update endpoint authorizes by the path ID only, with no session-to-object ownership check. Separately, the registration endpoint returns the existing user's lead `id` inside a "lead already verified" error when an existing email is submitted, turning email knowledge into object ID disclosure.
- **Exploitation steps:**
  1. POST to `/api/leads` with the victim's email; the error response for an already-registered lead includes that lead's `id`.
  2. Use that id in `PUT /api/lead/<id>/phone-number` with an attacker-chosen number, from the attacker's own session.
  3. Responses from the combined flow also exposed the victim's personal details.
- **Bypasses used:** ID unguessability bypassed via the information leak in the signup error, not by guessing.
- **Impact:** Change any user's phone number and read their PII, given only their email address.
- **Bounty:** Not stated in source.
- **Lesson / pattern:** Random IDs only raise the cost of discovery; hunt for the leak (signup, reset, invite, share flows) and note every odd behavior for chaining.
- **Source:**
  - Title: Tackling IDOR on UUID based objects
  - Author: Veshraj Ghimire (Pentester Nepal)
  - URL: https://medium.com/pentesternepal/tackling-idor-on-uuid-based-objects-71e8cb2dc265
  - Date: 2024-01-31
  - Disclosed report: none
  - Raw file: `sources/idor/12-tackling-idor-uuid-objects.md`

Tags: uuid, enumeration, pii-disclosure, api
