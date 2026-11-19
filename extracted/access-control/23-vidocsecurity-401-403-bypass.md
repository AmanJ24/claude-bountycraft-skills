# The 401/403 bypass arsenal: when an endpoint is blocked, change the method, header, path form, protocol, or case

- **Vuln type / sub-variant:** access-control bypass of 401/403 on a restricted endpoint via request-shape manipulation (reference methodology); not a single bug but the toolkit for defeating front-end or misconfigured path/IP restrictions
- **Target type:** any endpoint returning 401/403 (admin panels, internal paths) behind a proxy, WAF, or framework route guard
- **Recon signal:** a suspicious endpoint (e.g. `/admin`) answers 401 or 403; the question is whether the block is enforced consistently across methods, headers, path encodings, and protocol versions
- **Root cause:** access control applied inconsistently: enforced for one method/path-form/IP but not others, or applied at a proxy that can be tricked into forwarding a request the backend then serves
- **Exploitation steps (techniques to try, roughly in order):**
  1. HTTP method/verb fuzzing: resend as POST, PUT, PATCH, HEAD, OPTIONS, or a non-existent verb (`FOO`); a 403 on GET may be 200 on POST.
  2. Header fuzzing: add `X-Forwarded-For`, `X-Real-IP`, `X-Forwarded-Host`, `X-Original-URL`, `X-Rewrite-URL` etc. with `127.0.0.1` / `localhost` to defeat IP-based filters.
  3. Path fuzzing and creative literals: `/%2e/admin`, `/%252e/admin`, `/..%00`, `/%20/`, trailing-dot and double-encoding to make the server resolve a different path than the guard matched.
  4. Protocol downgrade: HTTP/2 blocked? try 1.1, or 1.0 with the Host header removed for unexpected behaviour.
  5. Case switching: `/User`, `/USer`, `/%75ser` against case-sensitive guards.
  6. Hop-by-hop header abuse: list a header in `Connection:` so a proxy strips it before the backend, which may have relied on it for the auth decision.
  7. Framework-specific: Spring < 5.3 `useSuffixPatternMatch` means `/admin` also matches `/admin.anything`.
- **Bypasses used:** verb tampering, IP-spoof headers, URL-encoding/path normalization discrepancies, protocol downgrade, case variance, hop-by-hop stripping, suffix matching
- **Impact:** reaching admin/internal functionality that a 401/403 was meant to protect; the severity is whatever the unblocked endpoint exposes
- **Bounty:** not stated (reference article)
- **Lesson / pattern:** a 401/403 is a claim, not a guarantee; enforce authorization identically across every method, path encoding, header, and protocol version, at the application layer rather than only at a proxy or route prefix. Removing a header or re-encoding a path should never change the access decision.
- **Source:**
  - Title: Ultimate 401 and 403 bypass methods
  - Author: Klaudia Kloc (Vidoc Security Lab)
  - URL: https://blog.vidocsecurity.com/blog/401-and-403-bypass-how-to-do-it-right
  - Date: 2023-05-09
  - Disclosed report: none
  - Raw file: `sources/access-control/23-vidocsecurity-401-403-bypass.md`

Tags: 403-bypass, 401-bypass, method-fuzzing, header-fuzzing, path-normalization, double-encoding, protocol-downgrade, hop-by-hop, case-switching, spring, methodology
