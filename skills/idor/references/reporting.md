# Reporting IDOR

Lead with the broken check and the impact rung, proven with two accounts you control.

## Template

**Title:** `IDOR in <feature> via <reference> allows <action> of other users' <object> (<impact>)`
e.g. "IDOR in profile update via body `id` allows changing any user's email, leading to account takeover".

**Summary (2-3 sentences):** which endpoint, which client-supplied reference, what the server fails to check, and the impact rung reached.

**Steps to reproduce (two-account PoC):**
1. Create account A (attacker) and account B (victim). State B's identifier (from B's own session: you own it).
2. The exact request from A, with B's identifier swapped in (path/body/header), and A's session/cookie.
3. The response showing B's data or the changed state. Redact B's real values if B is not a pure test account.
4. For escalation, the follow-on request (e.g. forgot-password to the changed email) and the result.

**Root cause (name the exact missing check):**
- "The handler selects the record by the body `id` and never compares it to the authenticated session user."
- "Authorization is checked against the object in the URL but the action is applied to the `owner_id` in the body."
- "The error serializer returns the object under `context.subject` even on a 422 deny."
- "A revoked/inactive object is still resolvable by its UUID."

**Id reachability:** how an attacker obtains the identifier: sequential/predictable (show the stride), or leaked (archive URL, JWT payload, error message, schema gap). This is what makes it exploitable rather than theoretical.

**Impact:** the rung reached (read / update / delete / takeover / mass or unauth) and the blast radius (one user / all users / all tenants). Tie to concrete loss: PII, payment data, account control, data destruction. Suggest a CVSS with justification.

**Remediation:** enforce per-object authorization on the server for every request, deriving the subject from the session, not from client input; check object-to-tenant and object-to-owner on every code path and HTTP method, including batch and new endpoints; do not include object data in denied responses; treat unguessable ids as defense-in-depth, not authorization.

**Scope note:** confirm in scope; state that you used test accounts and stopped at proof.

## What raised trust and severity in the set
- **Two-account proof** with the victim id and the response (every strong report).
- **Climbing the ladder** and demonstrating the top rung (change-email shown through to login; write shown through to leaked secret). (notes 01, 06)
- **A concrete attacker id-harvest path**, which converted an N/A ("you need the UUID") into a valid report. (note 18)
- **Reading the denied body** and showing the object came back despite the 403/422. (notes 08, 06)
- **Explicitly stopping at proof** and not enumerating real users. (note 02)

## What to avoid
- Reporting a client-value change (price, quantity, role) as IDOR; that is business-logic/mass-assignment, and needs downstream proof. (note 07)
- Claiming IDOR when the object is your own, is public by design, or the id is properly ownership-checked.
- Including real victims' PII, payment data, or full dumps. Redact; use test accounts.
- Stopping at a changed UI value without proving the server-side/downstream effect. (note 07)
- A bare 403 with no data back and no working bypass, reported as if it were access.
