# chs.us: authentication bypass taxonomy (JWT, OAuth, SAML, session), "an identifier is not a credential"

- **Vuln type / sub-variant:** reference taxonomy of authentication bypass classes: injection-based login bypass (SQLi/NoSQL), HTTP parameter pollution, session fixation/hijacking, OAuth (redirect_uri, state, client-id confusion), SAML (signature wrapping, comment injection), JWT (alg=none, RS256->HS256 key confusion, kid injection, weak HMAC), and fail-open control-flow bugs
- **Target type:** login forms, session managers, OAuth/SAML/JWT verification backends, multi-tenant consoles
- **Recon signal:** a login decision that depends on a client-supplied value: a hashed password field with a controllable type, a JWT whose header selects the algorithm, a `kid` used in a path or query, an OAuth `redirect_uri`, or auth code with branches that lack a final `else`
- **Root cause:** authentication trusted to the client or left to fail open: the server reads the algorithm from the token instead of pinning it, uses its RSA public key as an HMAC secret, loads a key from an attacker-controlled `kid`, regenerates no session id after login, or assigns no interceptor to a request that passes every check
- **Exploitation steps (the JWT set tested first, in order of how often they still land):**
  1. alg=none: set header `{"alg":"none"}`, change `sub` to the target user, drop the signature; libraries that honour the header accept it.
  2. Key confusion: switch header to HS256 and sign with the server's RSA public key as the HMAC secret; a verifier honouring the header checks out.
  3. kid injection: point `kid` at `/dev/null` or a value you control and sign with that known key.
  4. Weak secret: crack a short/default HS256 secret offline with hashcat.
  5. Fail-open (Veeam VSPC CVE-2026-58073): present a self-signed cert carrying a connected agent's GUID; the code trusts knowing the GUID as being the agent, and a request passing the check gets no interceptor, landing on the full agent surface.
- **Bypasses used:** header manipulation, key substitution, parameter pollution (`password=wrong&password=correct`), session id reuse, identifier-as-credential
- **Impact:** full account takeover, privilege escalation, cross-tenant access, and in the Veeam chain unauthenticated RCE on a console above every tenant's backups
- **Bounty:** not stated (reference guide; cites HackerOne-sourced Veeam CVEs)
- **Lesson / pattern:** auth bypasses rarely look like broken crypto; they look like an identifier trusted as a credential or a branch that fails open because nobody wrote the `else`. Pin the JWT algorithm and audience server-side, regenerate sessions on login, strictly validate redirect_uri, and trace the request that satisfies every condition.
- **Source:**
  - Title: Authentication Bypass Security Guide
  - Author: Carl Sampson (chs.us)
  - URL: https://chs.us/guides/authentication-bypass/
  - Date: 2026-05-02 (updated 2026-09-12)
  - Disclosed report: none
  - Raw file: `sources/access-control/06-chs-authentication-bypass-guide.md`

Tags: auth-bypass, jwt, alg-none, key-confusion, kid-injection, oauth, saml, session-fixation, fail-open, identifier-as-credential, taxonomy
