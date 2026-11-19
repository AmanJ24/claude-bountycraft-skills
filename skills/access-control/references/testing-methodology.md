# Testing methodology for access control and auth bypass

Step-by-step behind SKILL.md section 3. Research/methodology only: run this only against an authorized, in-scope program with accounts you own. The core discipline (notes 03, 22): **capture what a privileged identity can do, then try to do it as an unprivileged one, and confirm server-side.**

## 0. Set up identities

- Hold at least two accounts: a **low-privilege** one (attacker) and a **victim/high-privilege** one (to define the boundary and to read back results).
- In a multi-tenant app, create **two tenants/orgs** you own (the Auth0 cross-tenant bug, note 17, needed an attacker tenant).
- Keep separate browser profiles / Burp sessions so you do not mix tokens.

## 1. Map roles and objects

- Browse the app fully as the **privileged** account with the proxy recording. Every admin-only action you take is now a captured request (method, path, headers, token, body).
- Enumerate hidden surface the UI will not show you:
  - JS bundles and **source maps** (reveal routes, role strings, feature flags, storage keys).
  - **Swagger/OpenAPI**, GraphQL introspection, `/config.js`, `robots.txt`, sitemaps.
  - Proxy history for calls fired on tab-switch or load that the UI gates (note 21).
- Write down, per object (user, order, team, tenant, balance), which endpoints **read** it and which **act on** it, and what role each nominally requires.

## 2. Vertical: replay privileged actions as a lower role (MFLAC)

- For each admin-capable request, swap in the **low-privilege** account's token/cookies and resend unchanged.
- A `200`/success means no server-side role check. Expect this most on:
  - **Create / Copy / Assign** operations (note 21).
  - **Self-targeted** role/privilege fields: set `role`, `isAdmin`, `privilege`, `team_leader_ids:[me]`, `copy_from` (notes 08, 21).
  - Methods the UI never issues: `PUT`, `POST`, `DELETE`, `PATCH` to a resource that only serves `GET` in the UI.
- Also test **unauthenticated** replay of authenticated actions (drop the token entirely); SPAs that gate only client-side often have backends that never validate the bearer token.

## 3. Horizontal: swap identifiers for another identity

- Change `user_id`, `account_id`, `org_id`, `team_id`, `phone`, `invoice_id` to a **second account you own** and look for its data or the ability to act on it (note 15).
- Decode opaque cookies / `state` / tokens first; the id you need to change may be inside a base64 JSON blob (note 14).
- For deep object-id technique (enumeration, GUIDs, GraphQL node ids, mass assignment), use the `idor` skill; this skill's interest is the authorization gap, not the id format.

## 4. Force-browse hidden and role-gated routes

- Request `/admin/*`, `/manage/*`, `/internal/*`, numeric/named sub-resources, and multi-step endpoints **directly**, as a lower or unauthenticated role.
- Try the **middle of a multi-step flow** out of order (a registration step that sets a role, a portlet id, note 07; note 11).
- Include methods the UI never sends to those routes.

## 5. Run the 401/403 bypass arsenal

On anything that answers forbidden, work through `references/403-401-bypass.md`: method/verb, IP/URL-override headers, path normalization and double-encoding, case switching, protocol downgrade, hop-by-hop stripping, framework quirks. Confirm you received the **actual** protected content, not a soft-200 or login page.

## 6. Attack authentication where a token or SSO guards the action

Use `references/oauth-jwt-auth-bypass.md`:
- **JWT:** decode it; test `alg=none`, RS256->HS256 key confusion, `kid` injection, an unverified signature (edit a claim and resend), and whether a mutable `email`/`sub` is trusted as identity.
- **OAuth/OIDC:** test `redirect_uri` validation, `state` presence/validation, PKCE enforcement, provider mix-up, scope upgrade on the code exchange, `acr`/`amr` downgrade, and alternate grants (ROPC) that skip verification.
- **Session/SSO:** session fixation (no new id on login), leaked privileged-session ids, host-header poisoning, SSO responses that can be tampered (302->200, Location dropped, note 07).

## 7. Re-check after state changes

- Demote or remove a user, then confirm their **old session/token** actually loses access. Many apps never revoke (note 22).
- After an email/role change, re-verify what the changed account can now do.

## 8. Confirm server-side, then stop at proof

- A change is only real when a **privileged read-back** (as the victim/admin, or via a profile endpoint) shows the server persisted it. A reflected or client-only change proves nothing (see SKILL.md section 5).
- Demonstrate **one** unauthorized action or read on accounts you own. Do not pivot into other real users' data and do not make destructive admin changes.

## Tooling quick list

- **Burp proxy + Repeater**: capture, token-swap, replay, method/header fuzzing. Match-and-replace rules to automate a header/method swap across traffic (note 07).
- **Burp Intruder / ffuf**: force-browse route lists and 403-bypass header/path wordlists.
- **jwt_tool / jwt.io / token.dev**: forge and tamper JWTs (notes 06, 24). Keep forged tokens on test accounts only.
- **Decoder**: base64/URL-decode cookies and `state` to find ids and redirect paths.
- Discovery: `curl -s --http2 ...`, source-map extractors, OpenAPI/Swagger readers.
