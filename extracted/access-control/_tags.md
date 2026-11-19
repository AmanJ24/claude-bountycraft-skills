# Canonical tags (Access Control / Auth Bypass)

Normalized 2026-11-19. Use these spellings in `Tags:` lines.

- Sub-variant (authz): vertical-privesc, horizontal-privesc, horizontal-to-vertical, mflac (missing function-level access control), idor, multi-tenant, cross-tenant, multi-step, registration-privesc
- Vector (authz): front-end-only (UI hides, API allows), api-authz, hidden-endpoint, unprotected-admin, security-by-obscurity, parameter-role, mass-assignment-role, self-assign-role, query-parameter, force-browsing, request-replay, copy-operation, referer-based, method-based, url-based
- 403/401 bypass: header-bypass, header-fuzzing, method-fuzzing, method-override, path-normalization, double-encoding, case-switching, protocol-downgrade, hop-by-hop, spring
- Auth bypass (identity): auth-bypass, session-fixation, session-leak, session-id-exposed, session-not-enforced, session-poisoning, cookie-replay, impersonation, identifier-as-credential, client-trusted-value, client-side-enforcement, client-side-2fa, fail-open, empty-password, sso-misconfig, response-tampering, static-token-auth
- JWT: jwt, alg-none, key-confusion, kid-injection, jwt-audience, jwt-claim-trust, signature-not-verified, forged-token, cwe-284
- OAuth / OIDC: oauth, oidc, redirect-uri, open-redirect, redirect-allowlist, state-csrf, pkce-downgrade, provider-mixup, code-binding, authorization-code-leak, access-token, token-leak, implicit-grant, implicit-flow, scope-upgrade, ropc, email-verification-bypass, email-claim, email-account-assignment, mutable-claims, client-confusion, host-header-poisoning, acr-amr-2fa-bypass, dynamic-client-registration, collaborator, idp, noauth-class
- Root cause: missing-server-check, trust-user-input, otp-not-binding, multi-tenant-isolation, alternate-grant-flow, mfa-adjacent
- Impact: account-takeover, admin-access, admin-takeover, privilege-escalation, data-exposure, broken-access-control
- Chain / cross-ref: ssrf, path-traversal, access-control-chain, admin-bot-confused-deputy, ctf
- Meta: methodology, taxonomy, reference, research, defense, source-partial, informative-dispute, cve

Notes:
- `missing-server-check` + `front-end-only` are the near-universal authz root-cause tags: the UI hides it, the server does not enforce it. The spine question is "what on the server enforces this, and is it the UI or nothing?".
- Scope split: this skill owns authorization-enforcement failures (vertical/horizontal privesc, MFLAC, 403/header/method bypass) and authentication bypass (JWT, OAuth/OIDC, session). Pure object-level IDOR is cross-referenced to the `idor` skill; flawed-state-machine auth bypass to `business-logic`; TOCTOU/refresh-token races to `race-conditions`.
- References / taxonomy sources (no single finding): 01 (PortSwigger spine), 02 (HackTricks 403/401), 03 (HackerOne BAC methodology), 04 (HackTricks OAuth-to-ATO), 05 (0xn3va OAuth), 06 (chs auth-bypass), 07 (Synack 5 techniques), 22 (Asfan privesc playbook), 23 (Vidoc 403 bypass). These carry `methodology`/`taxonomy`/`reference`.
- Real disclosed findings: 08-21, 24 (H1, Medium/InfoSec, Sentor, huntr). 15/20 are partial scrapes (`source-partial`). 16 is an `informative-dispute` (program ruled it a bug, not a vuln). 19 (t4nv1 OAuth redirect_uri prefix-match + open-redirect ATO, $8k) was user-supplied from `Manual_downloads/` and is now extracted.
- OAuth and JWT are deep sub-areas represented here but not exhausted; each could become its own skill later. Flag this in SKILL.md.
