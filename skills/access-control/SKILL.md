---
name: access-control
description: Use when testing whether a user can act outside their intended permissions: reaching an admin or privileged function as a lower role (vertical privesc), acting on another user's or tenant's object (horizontal privesc), calling an API the UI hides (missing function-level access control), bypassing a 401/403 by changing the method, header, path encoding, or protocol, or defeating authentication itself through JWT forging (alg=none, key confusion, unverified signature), OAuth/OIDC flaws (redirect_uri theft, state/CSRF, mutable email claim, ROPC), or session and SSO misconfiguration. Covers recognizing where authorization is delegated to the client, the probe methodology, the bypass arsenal, false positives, and reporting. Research/methodology only: it does not send requests to live targets.
---

# Access Control / Auth Bypass

Synthesized from 24 real bug bounty findings and references (OWASP A01, the PortSwigger Access Control topic and labs, HackTricks 403/401 and OAuth-to-ATO, the 0xn3va OAuth cheat sheet, Synack and chs.us auth-bypass roundups, the Vidoc 403-bypass guide, and disclosed HackerOne/huntr/Sentor/Medium reports spanning read-only-to-admin, header-bypass admin access, cross-tenant Auth0 impersonation, GitLab ROPC verification bypass, JWT account takeover, and an $8k redirect_uri-bypass ATO). Every pattern traces to `sources.md`. Disclosed impact ran from Medium (horizontal data access) to Critical (9.8, forged-JWT account takeover); the money is in reaching a privileged function or another user's data that the server never re-checks.

**The one idea:** access control is an answer the **server** must give to one question, on **every** request: *is this caller allowed to perform this action on this object?* A bug exists wherever that answer is delegated to something the attacker controls: the UI (which only hides the button), a hidden or guessable route, a client-supplied id (`?user_id=`, `account_id` in a cookie), a header or method the guard did not cover, or a token/claim the attacker can forge or set (a JWT signature never verified, an `email` claim trusted as identity). To find these, **name the resource and the action, then ask what on the server authorizes this caller for it**. If the honest answer is "the UI", "a hidden route", "a value the client sent", or "nothing", there is a bug.

**Scope rule:** methodology and guidance only. It does not fire requests at live targets. Real testing needs an authorized, in-scope program and at least two accounts you control (a low-privilege one and a victim/high-privilege one); escalating privileges and reading other users' data crosses into real unauthorized access if done off-scope, so stay on accounts and tenants you own and stop at proof (see `../../CLAUDE.md`).

**Scope split (what lives elsewhere):** pure object-level reads by swapping an id are shared with the `idor` skill (cross-reference, not duplicated here); flawed-state-machine / forced-browsing-past-a-step auth bypass is in `business-logic`; TOCTOU and refresh-token races are in `race-conditions`. This skill owns **authorization enforcement** (vertical/horizontal privesc, missing function-level access control, 401/403 bypass) and **authentication bypass** (JWT, OAuth/OIDC, session, SSO). OAuth and JWT are deep enough to each become their own skill later; this covers the account-takeover-grade essentials.

## 1. Recognize it

Broken access control is the request that *looks* authorized because the client was allowed to build it, but the server never confirmed the caller may act on that target. The input is usually a legitimate, well-formed request; the bug is that it succeeds from the wrong account, role, method, or tenant.

Signals, grouped by the four ways authorization leaks (full catalog in `references/recognizing-and-surfaces.md`):

- **The UI hides it, the API allows it (MFLAC).** A page or button is absent for your role, but the underlying API call still works with your token. The classic tell is "Only editors can do this" in the UI while `POST`/`PUT` to the endpoint returns 200 (read-only-to-admin via API PUT, note 08; Scheduler-to-team-admin, note 21).
- **A client-supplied identifier names the target.** The account acted on is taken from a URL param, body field, or cookie (`?phone=`, `?user_id=`, `account_id` in a base64 cookie) rather than the session. Swap it for another user's value (MTN `?phone=` horizontal access, note 15; and see the `idor` skill).
- **A hidden, guessable, or role-gated route is reachable directly.** `/admin/*`, `/manage/*`, `/internal/*`, a numeric CMS portlet id, or a registration step that assigns a role. Force-browse to it as a lower role (Liferay portlet self-registration, note 07; registration privesc, note 11).
- **A 401/403 that is not enforced consistently.** The same resource answers differently to a different HTTP method, an `X-Original-URL`/`X-Forwarded-For` header, a re-encoded path, or a downgraded protocol (Clario `X-Rewrite-URL` admin access, note 10; the 403-bypass arsenals, notes 02, 23).
- **Authentication itself is forgeable or skippable.** A JWT whose signature is never verified or uses `alg=none`/key confusion; an OAuth `redirect_uri` that leaks the code/token (even via a loose prefix match chained to a same-domain open redirect, $8k ATO, note 19); a mutable `email` claim trusted as identity; an alternate grant (ROPC) that skips verification the web login enforces (forged-JWT ATO, note 24; OAuth redirect_uri token theft, note 18; GitLab ROPC, note 16; Auth0 cross-tenant, note 17).

## 2. Where it hides

Feature classes that actually produced findings. Full list with examples in `references/recognizing-and-surfaces.md`.

- **User/role management and profile endpoints**: self-targeted updates of a `role`, `isAdmin`, `privilege`, or `team_leader_ids` field; the API accepts the change the UI would not offer (notes 08, 09, 11, 21).
- **Admin and internal routes**: `/admin`, `/manage`, `/internal`, settings, user lists, impersonation/"act as user", support login. Reachable by force-browsing, a header bypass, or a leaked privileged session (notes 10, 12, 14).
- **Multi-tenant / org-scoped resources**: `/org/1/users` -> `/org/2/users`; a token or session from tenant A accepted at tenant B (note 17; note 22 playbook).
- **OAuth/OIDC login flows**: `redirect_uri`, `state`, `response_type`/`response_mode`, the token/code exchange, account-linking by email, and ROPC/alternate grants (notes 04, 05, 16, 18, 20).
- **JWT-authenticated APIs**: anywhere a bearer JWT authorizes the action; test whether the server verifies issuance, signature, algorithm, audience, and expiry (notes 06, 17, 24).
- **401/403 endpoints surfaced in recon**: any interesting path that answers "forbidden"; it is a candidate for the bypass arsenal (notes 02, 23).
- **Registration, invites, and "temporary" elevation**: a signup step that sets privileges, reusable invite/impersonation links, access that persists after demotion (notes 11, 22).

## 3. Test methodology

Work from two accounts and a simple rule: capture what the privileged account can do, then try to do it as the unprivileged one. Full step-by-step in `references/testing-methodology.md`.

1. **Map roles and objects.** List the roles you can hold, and for each privileged action note the exact request (method, path, body, token). Burp proxy history during a privileged session is the raw material; so are JS bundles, source maps, and Swagger/OpenAPI, which disclose hidden routes and role strings even when the UI hides them.
2. **Replay privileged actions as a lower role (vertical).** Take each admin-capable request, swap in the low-privilege account's token/cookies, and resend. A 200 means no server-side role check (MFLAC). Pay special attention to Create, Copy, and Assign operations and to self-targeted role/privilege fields.
3. **Swap identifiers for another user/tenant (horizontal).** Change `user_id`, `account_id`, `org_id`, `phone`, and similar to a second account you own; reading or editing its data is horizontal privesc. (Deep object-id technique: `idor` skill.)
4. **Force-browse hidden and role-gated routes.** Request `/admin/*`, `/manage/*`, numeric or named sub-resources, and multi-step endpoints directly, as a lower or unauthenticated role, including `POST`/`PUT`/`DELETE` the UI never sends.
5. **Run the 401/403 bypass arsenal** on anything forbidden: method change, IP/URL-override headers, path normalization, case switching, protocol downgrade, hop-by-hop stripping (see `references/403-401-bypass.md`).
6. **Attack authentication** when a token or SSO flow guards the action: JWT forging, OAuth redirect_uri/state/mix-up, mutable-claim and alternate-grant bypass (see `references/oauth-jwt-auth-bypass.md`).
7. **Re-check after state changes.** Confirm a demoted or removed user's old session/token actually loses access; many apps never revoke it (note 22).
8. **Stop at proof** on accounts you own: demonstrate the one unauthorized read or action, do not pivot into other real users' data or make destructive admin changes.

## 4. Known bypass / evasion patterns

Summaries; the full arsenals are in `references/403-401-bypass.md` and `references/oauth-jwt-auth-bypass.md`.

- **401/403 request-shape bypass (notes 02, 10, 23):** change the method (`GET`->`POST`/`PUT`/`HEAD`/`FOO`); add `X-Original-URL`/`X-Rewrite-URL`/`X-Forwarded-For`/`X-Real-IP`; re-encode the path (`/%2e/admin`, `/%252e/`, trailing dot, double-encode, case switch); downgrade HTTP/2 to 1.1 or 1.0 with no Host; strip a relied-on header via `Connection:`; Spring `/admin` also matching `/admin.anything`.
- **Missing function-level access control (notes 08, 21):** the hidden API call works with a low-privilege token; replay it directly, including methods the UI never issues.
- **JWT forging (notes 06, 17, 24):** `alg=none` with the signature dropped; RS256->HS256 key confusion using the public key as the HMAC secret; `kid` injection; an unverified signature where any edited token is accepted; a mutable `email`/`sub` claim trusted as identity.
- **OAuth/OIDC (notes 04, 05, 16, 18, 20):** unvalidated or weakly-matched `redirect_uri` leaking the code/token; missing/static/unvalidated `state` for CSRF account-linking; PKCE downgrade; provider mix-up; scope upgrade on the code exchange; host-header poisoning; `acr`/`amr` downgrade for 2FA bypass; ROPC or another grant that skips the verification web login enforces; pre-account-takeover via unverified email.
- **Privileged-session and confused-deputy tricks (notes 12, 14):** a leaked impersonation session id replayed from a cookie; a UI-gated `/admin` action executed by a server-side admin bot that faithfully renders attacker-controlled page state.

## 5. Common false positives

- **A hidden UI with a server-side check.** The button is absent but the API returns 401/403 for your role: that is correct enforcement, not a bug. The finding requires the action to actually succeed.
- **A reflected or self-only change.** Editing your own role field in a response that the server ignores, or a client-side state change that no API trusts, proves nothing until a privileged read-back confirms the server honored it.
- **An "admin" page with no privileged function or data.** Reaching `/admin` that renders an empty shell or only your own data is not impact; you need a privileged action or another user's data.
- **A 403 bypass that returns the same 403 body with a 200 status, a login page, or an error.** Confirm you received the actual protected content or performed the action, not a soft-200 wrapper.
- **An auth "bypass" with an unrealistic precondition.** A flow that needs the victim's current password (GitLab ROPC, note 16, was closed Informative on exactly this) or a value only the victim can produce may be ruled a bug, not a vulnerability. State the precondition honestly.
- **An open redirect with no sensitive data in the URL.** It is only account takeover when the OAuth code/token actually rides the redirect (note 20); otherwise report it as a lower-severity open redirect.

## 6. Reporting

Lead with the crossed boundary and a two-account proof. Full guidance in `references/reporting.md`.

- **Name the invariant and who broke it:** "a Scheduler-role user performed an Editor-only action", "account A read account B's data", "an unauthenticated request reached `/admin`". State the exact request and the role/account it was sent from.
- **Prove the boundary with two identities:** show the privileged action succeeding from the low-privilege (or victim's-neighbor) account, ideally alongside the same request correctly denied in its intended context. A privileged read-back that confirms the server persisted the change is the strongest evidence.
- **Articulate real impact:** full account/tenant takeover, admin functions, other users' PII, financial actions. Map it to the concrete business function, and for auth-bypass note whether it is unauthenticated or needs a weak precondition.
- **Keep it on accounts you own and stop at proof:** one demonstrated unauthorized action, no pivoting into real users' data, no destructive admin changes.

## 7. Sources

See `sources.md` for the full list (title, author, URL, disclosed-report link, bounty) behind every claim above, keyed to the extracted notes in `extracted/access-control/`.
