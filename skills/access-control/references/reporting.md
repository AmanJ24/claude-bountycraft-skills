# Reporting access-control and auth-bypass findings

Full guidance behind SKILL.md section 6. Access control is OWASP A01 and the most-reported web class; triage hinges on a clean two-identity proof and an honest impact statement. The difference between a dismissed report and a Critical is whether you showed the **server** letting the wrong caller through.

## Structure

1. **Title: the crossed boundary, concretely.** "Scheduler-role user performs Editor-only team creation via API", "Account A reads Account B's statements via `?phone=`", "Unauthenticated request reaches `/admin` via `X-Rewrite-URL`", "Forged JWT takes over any account". Name the role/account and the action.
2. **Affected endpoint and the exact request.** Method, path, headers, token/cookie used, and body. Show the request as sent from the **wrong** identity.
3. **Two-identity proof.** The strongest evidence is a matched pair:
   - the privileged action **succeeding** from the low-privilege (or victim's-neighbor, or unauthenticated) identity, and
   - the same request **correctly denied** in its intended-but-wrong context, or a privileged **read-back** (as the victim/admin, or a profile endpoint) confirming the server persisted the change.
   A reflected or client-only change is not proof; the server must have honored it.
4. **Impact, mapped to the business.** State what the boundary break yields: full account/tenant takeover, admin functions, other users' PII, financial actions, data modification/destruction. For auth bypass, say whether it is **unauthenticated** or needs a precondition.
5. **Remediation.** Enforce authorization server-side on every request, keyed to the authenticated session/principal, not to the UI, a hidden route, a client-supplied id, or a mutable/forgeable claim. For JWT: verify signature/alg/aud/iss/exp. For OAuth: exact-match redirect_uri, enforce state/PKCE, bind identity to iss+sub.

## Severity framing (what moved the needle in this set)

- **Critical:** unauthenticated account takeover (forged JWT, note 24, 9.8), cross-tenant impersonation (note 17), full admin via privilege escalation (note 12, 9.0).
- **High:** MFLAC granting admin-like control (notes 08, 21), header bypass to admin (note 10), OAuth token/code theft -> ATO (notes 18, 20).
- **Medium:** horizontal access to another user's data without further escalation (note 15, 6.5).
- Argue up from "I reached X" to "which means an attacker can do Y to Z users/money". A reachable admin function with a real action beats a reachable-but-empty admin page.

## Make impact undeniable

- **Chain to the worst primitive.** A Scheduler-creates-teams bug became structured privilege escalation once `copy_from` and `team_leader_ids` were shown (note 21). A horizontal flaw that leaks a password or session becomes vertical ATO (note 12).
- **Quantify reach.** "Any MTN number in Nigeria" (note 15), "any authenticated user, including external, becomes instance admin" (note 12). Scope the blast radius.
- **For multi-tenant**, show you crossed from your tenant into another you own, and name what a real victim tenant would expose.

## Honesty that protects your credibility

- **State preconditions plainly.** GitLab ROPC (note 16) was closed Informative because it needs the victim's password; the program disputed the root cause. If your bypass needs a leaked password, a same-locality tenant (note 17), or victim interaction, say so up front and argue impact within that constraint.
- **Do not overclaim an open redirect.** It is ATO only when the OAuth code/token actually rides the redirect (note 20); otherwise report the open redirect at its real (lower) severity.
- **Distinguish issuance bugs from verification bugs** (note 04): "the app signed my chosen identity" is different from "I forged a signature"; naming it correctly guides the fix and shows you understand it.

## Evidence hygiene and scope

- Use **accounts and tenants you own**; demonstrate one unauthorized action/read and stop. Do not pivot into real users' data, exfiltrate PII, or make destructive admin changes.
- Redact other users' data if it appears incidentally. Keep forged tokens bound to your test accounts.
- Record the request/response pair (and a short video for multi-step chains like note 14) so triage can reproduce without re-deriving the whole chain.
