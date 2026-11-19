# OAuth / OIDC / JWT / session authentication bypass

Full detail behind SKILL.md sections 1, 4, and 6. Synthesized from notes 04 (HackTricks OAuth-to-ATO), 05 (0xn3va OAuth), 06 (chs.us), 07 (Synack), 16 (GitLab ROPC), 17 (Auth0 cross-tenant), 18 (redirect_uri token theft), 19 (t4nv1 prefix-match + open-redirect $8k ATO), 20 (LINE open-redirect code leak), 24 (memos JWT). These are account-takeover-grade; keep all forging and tampering on accounts/tenants you own.

The unifying rule across both JWT and OAuth: **an identifier or claim is not a credential, and decoding is not verifying.** A bug exists wherever the server trusts a value the attacker can set or forge (an unverified signature, an `email` claim, a `redirect_uri`, a `username`) to decide who you are.

---

## Part A: JWT

When an app authorizes from a bearer JWT, the **signature check is the authentication**. Break or skip it and you become any user. Test first, roughly by how often they still land (notes 06, 24):

1. **Signature never verified (note 24).** Edit a claim (change `sub`/user id) at jwt.io/token.dev and resend. If accepted, the server decodes but never verifies it issued the token. This is a direct ATO (memos, CVE-2023-4696, Critical 9.8).
2. **`alg=none`.** Set header `{"alg":"none"}`, change claims, drop the signature. Libraries that read the algorithm from the token accept it. Try case variants (`None`, `nonE`) against case-sensitive blocklists (the historic Auth0 `nonE` bypass).
3. **Key confusion RS256 -> HS256.** Switch the header to HS256 and sign with the server's **RSA public key** as the HMAC secret. A verifier that honours the header uses the same public key and the signature checks out.
4. **`kid` injection.** The `kid` header selects the key; if used in a file path or SQL query, point it at `/dev/null`, a predictable file, or an injectable value and sign with that known key.
5. **Weak HMAC secret.** Crack a short/default HS256 secret offline (hashcat) and re-sign.
6. **Audience / issuer not checked (notes 07, 17).** A validly signed token is accepted though it was not issued for this app/tenant. Synack example: a sample Microsoft JWT accepted anywhere the signature was valid. Always test whether `aud`, `iss`, and `exp` are enforced.
7. **Mutable claim trusted as identity (notes 04, 17).** The app keys the user on `email`/`preferred_username` instead of stable `iss`+`sub`. Set `email=victim@...` (in your own tenant/IdP) and be treated as the victim.

Defense to cite: pin the algorithm, verify signature against the server's own key, enforce `iss`/`aud`/`exp`, and bind identity to `iss`+`sub`, never a mutable claim.

---

## Part B: OAuth 2.0 / OIDC

Grant recap: authorization-code (`response_type=code`, server exchanges code+secret for token) is the safe default; implicit (`response_type=token`/`id_token`, token in the URL fragment) is legacy and leak-prone.

### B1. redirect_uri weaknesses (notes 04, 05, 18, 20)
The `code`/token is appended to `redirect_uri`; if you can point it off-domain, you steal it. Probe:
- **No validation**: any absolute URL accepted (note 18 showed `redirect_uri=https://google.com` then a Collaborator host capturing the id_token).
- **Weak host matching**: `victim.com.evil.com`, `evil.com/victim.com`, `victimAvictim.com`, `victim.com@evil.com`, `victim.com#@evil.com`, IDN homographs (validated on punycode, redirected on Unicode).
- **Path tricks on an allowed host**: `/callback/../open-redirect?next=//evil`, an allowlisted domain that itself hosts an open redirect, XSS, or user content (note 20: open redirect inside the OAuth flow leaked the LINE code -> token).
- **Loose prefix match + same-domain open redirect (note 19, $8k ATO)**: when the IdP accepts any redirect_uri that merely *starts with* the registered callback, keep the prefix and append a traversal into a same-domain open redirect: `redirect_uri=https://app.com/auth/callback/../../redirect?url=https://attacker.com`. The prefix match passes; the client router resolves the traversal to `/redirect` and forwards the code to the attacker. Full domain and subdomain swaps are rejected, so test the traversal form specifically. No PKCE means the stolen code is directly redeemable.
- **Directory/traversal**: `/oauth/*` bypassed with `/oauth/../anything`.
- **Wildcard subdomain / dangling DNS**: `*.victim.com` + a takeoverable subdomain.
- **HPP**: send `redirect_uri` twice.
- Also probe `client_uri`, `policy_uri`, `tos_uri`, `logo_uri`, `jwks_uri`, `sector_identifier_uri` (the last few can trigger SSRF at registration; cross-ref `ssrf` skill).

### B2. state / CSRF (notes 04, 05)
`state` is the flow's CSRF token. If it is **missing, static, optional, or unvalidated**: complete your own OAuth flow, keep the final `?code=` request without sending it, then force the victim's browser to load it. Their account links to your IdP identity -> you log in as them. Test by removing `state`, tampering the returned value, or reusing a static one.

### B3. code/token binding (note 04)
- **Everlasting / replayable code**: redeem a captured code after minutes, or twice; a second token means clone-able sessions.
- **Code not bound to client**: redeem app A's code at app B's token endpoint; a token means broken audience binding (authorization-code swap).
- **Concurrent redemption**: race two token requests (cross-ref `race-conditions`).
- **PKCE downgrade**: strip `code_challenge`, get an unbound code, inject it into the victim's pending callback if the token endpoint ignores the verifier.
- **Provider mix-up**: when `state`/`nonce`/PKCE live in provider-agnostic cookies, a value minted for provider A satisfies provider B's callback; validate the authorization-response `iss` (RFC 9207).

### B4. token leakage (note 04)
Access tokens must never reach the resource owner's browser. Check for tokens in URLs/history/`Referer`, `localStorage`/`sessionStorage`, JS state, or a telemetry `postMessage` confused deputy. Implicit-grant apps that POST `{user_id, access_token}` to their own server to create a session (note 05): change `user_id` to the victim; if the server does not confirm the token belongs to that user, you impersonate them.

### B5. scope / acr / amr (notes 04, 05)
- **Scope upgrade**: add extra `scope` on the code exchange; if not validated against the original authorization, a higher-privileged token is issued.
- **`acr_values`/`amr_values` downgrade**: if the server honours them, request `amr_values=pwd` instead of `pwd+otp` to bypass 2FA.

### B6. account linking / pre-ATO by email (notes 04, 05)
- **No email verification on signup**: pre-create an account with the victim's email; when they later use social login, the app links it to yours.
- **IdP with lax verification**: the provider returns an unverified `email=victim@...`; a relying party that links by email alone binds the victim's local account. Check whether the RP honours `email_verified=false` and keys on `iss`+`sub`.

### B7. alternate grants and SSO flaws (notes 07, 16, 17)
- **ROPC (note 16)**: the `/oauth/token` password grant may skip verification the web login enforces (GitLab: new-device email verification bypassed; note the program ruled it Informative because it needs the victim's password, state such preconditions honestly).
- **Cross-tenant (note 17)**: a session/cookie from your tenant accepted at another tenant's `/authorize`, which signs a token for the victim's email. Validate which tenant+client a session is valid for.
- **Username-only SSO bridge (note 04)**: an unauthenticated `ssoAuthentication`/session-bootstrap endpoint that takes a `username` and signs a session without validating any IdP response. Call it with a victim username; a returned token is an auth bypass (a correctly signed token does not make it safe, the app signed the attacker's chosen identity).
- **SSO response tampering (note 07)**: flip `302`->`200` and drop `Location` to render a leaked internal response; reuse a sample/example token; reach a CMS create-account portlet directly.

### B8. consent and interaction (note 04)
- `prompt=none` to skip interactive confirmation when the user is already logged in.
- `response_mode` selection (`query`/`fragment`/`form_post`/`web_message`) to change where the code lands and enable postMessage/iframe leakage.
- **Clickjacking the consent dialog** if it lacks `X-Frame-Options`/`frame-ancestors`.

---

## Part C: session / cookie

- **Session fixation**: no new session id issued on login; set a known id pre-auth and reuse it.
- **Leaked privileged session (note 12)**: an impersonation/sudo session id displayed to a lower-privilege view, replayed from a cookie, then "Stop impersonating" lands you as admin.
- **Host-header poisoning (note 05)**: poison `Host` so the OAuth code/token (or a reset link) is sent to your domain (Periscope ATO class).
- **Tokens/identity in `localStorage` trusted by the API (note 04)**: overwrite `username`/`role`/`isAdmin`/tenant id and reload; any privilege change means the server trusts client state.

## Testing checklist (condensed)

1. Decode every JWT and `state`/cookie; note what identity claim the app keys on.
2. JWT: unverified-signature edit, `alg=none`/case, RS256->HS256, `kid`, aud/iss/exp, mutable claim.
3. OAuth: redirect_uri matrix, state removal/tamper, code replay/cross-client, PKCE strip, scope/amr upgrade, email linking, ROPC/alternate grant, prompt/response_mode.
4. Session: fixation, leaked privileged id, host-header, localStorage trust.
5. Always confirm with a privileged read-back on an account you own; state any precondition (needs victim password, needs same locality) honestly.
