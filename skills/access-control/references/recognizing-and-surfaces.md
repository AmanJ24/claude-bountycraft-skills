# Recognizing access-control bugs and where they hide

Full catalog behind SKILL.md sections 1 and 2. Every item cites an extracted note in `extracted/access-control/`.

## The four ways authorization leaks

Every finding in this set reduces to the server trusting something the attacker controls for the access decision.

1. **Delegated to the UI (missing function-level access control, MFLAC).** The front end hides a page or button for your role, but the API behind it enforces nothing. The gap is the difference between "cannot see it" and "cannot do it".
   - Read-only user edits own privileges via API PUT; UI hides the users page, the API does not check (note 08).
   - Scheduler role performs Editor-only team create/copy/assign via direct API call with a Scheduler token; UI says "Only editors can create teams", API returns 200 (note 21).
   - General playbook: capture an admin-only action, replay with low-privilege cookies; success means no server-side role check (note 22).

2. **Delegated to a client-supplied identifier.** The object acted on comes from a request value, not the authenticated session.
   - MTN offers dashboard keyed on `?phone=`; swap the number for horizontal access to any subscriber after a single OTP (note 15).
   - H1-2006 CTF: `account_id` in a base64 cookie concatenated into an internal URL (note 14).
   - Object-level id swapping in general: see the `idor` skill (deliberately not duplicated here).

3. **Delegated to a hidden or role-gated route.** Security by obscurity, or a check present on one path and absent on a sibling.
   - Liferay CMS portlet reachable by numeric `p_p_id=58`, exposing Create Account self-registration behind a single public login page (note 07).
   - Insufficient access control on registration escalates to Administrator (note 11).
   - Rocket.Chat guest escalates into the admin group via improper ACL (note 09).
   - Dust: privilege escalation in the edit/role path (note 13).

4. **Delegated to a forgeable or settable credential/claim.** Authentication output the attacker can produce.
   - usememos/memos never verifies it issued the JWT; any edited token takes over any account (note 24, CVE-2023-4696).
   - Auth0 cross-tenant: `/authorize` signs a valid JWT for a user that exists only in another tenant; apps trusting the `email` claim are impersonated (note 17).
   - OAuth `redirect_uri` leaks the id_token to an attacker host (note 18); open redirect in the OAuth flow leaks the code, then the token (note 20).
   - GitLab ROPC `/oauth/token` skips the new-device email verification web login enforces (note 16).

## Vertical vs horizontal vs MFLAC (name it precisely)

- **Vertical privilege escalation:** acting as a higher role (user -> admin), or unauthenticated -> authenticated. Notes 08, 09, 11, 12, 13, 14, 21.
- **Horizontal privilege escalation:** acting as a same-level different user/tenant. Note 15; `idor` skill for object-level.
- **Horizontal-to-vertical:** reading a higher-privileged user's secret (password, session) through a horizontal flaw, then logging in as them. Note 12 (leaked impersonation session -> admin); see `idor` password-disclosure chains.
- **Missing function-level access control:** the function exists and is callable; only the UI gates it. Notes 08, 21. This is the single most common and most reliably-paid class in the set.

## Feature-class surface map

| Surface | What to try | Notes |
|---|---|---|
| User/role/profile management | Self-target a `role`/`isAdmin`/`privilege`/`*_leader_ids` field via API | 08, 09, 11, 21 |
| Admin / internal routes | Force-browse `/admin`, `/manage`, `/internal`, settings, impersonation | 10, 12, 14 |
| Multi-tenant / org resources | Flip `org_id`/workspace id; replay a tenant-A token at tenant B | 17, 22 |
| OAuth / OIDC login | redirect_uri, state, response_mode, code/token exchange, email linking, ROPC | 04, 05, 16, 18, 20 |
| JWT-authenticated APIs | Verify issuance/signature/alg/audience/exp; forge and replay | 06, 17, 24 |
| 401/403 endpoints from recon | Method, header, path-encoding, protocol, hop-by-hop bypass | 02, 10, 23 |
| Registration / invites / elevation | Signup that sets a role; reusable invite/impersonation links; access after demotion | 11, 22 |
| Server-side bots / fetchers | Make an admin bot render attacker-controlled state that triggers a UI-gated action | 14 |

## Recon tells that an access-control bug is likely

- A JS bundle, source map, Swagger/OpenAPI, or `/config.js` that names routes, role strings, or storage keys the UI hides.
- A "disabled" or hidden control whose underlying endpoint is still visible in proxy history.
- An opaque cookie or `state` that base64-decodes to JSON with an `account_id`, `role`, tenant id, or `redirect` path (decode everything; base64 is not a secret).
- A 401/403 on an interesting path (worth the whole bypass arsenal).
- A login/SSO flow that offers both password and social sign-in for the same email (account-linking abuse).
- An identity value (username, email, phone, user_id) accepted in a request body where the session should have decided it.
