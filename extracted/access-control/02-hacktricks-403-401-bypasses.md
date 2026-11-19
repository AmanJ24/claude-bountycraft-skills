# HackTricks 403/401 bypass catalogue

- **Vuln type / sub-variant:** methodology/taxonomy (403/401 bypass techniques across method, header, path, parameter, protocol, and stale-authz classes)
- **Target type:** proxy-fronted or framework-routed endpoints returning 403/401; JSON APIs; ASP.NET Core apps; batched RPC endpoints
- **Recon signal:** a protected path answering 403/401 while a proxy sits in front, or one that behaves differently by method, case, or API version
- **Root cause:** parser or policy mismatch between proxy/WAF and backend, checks bound to raw strings, trust of client-supplied origin headers, authorization state cached or reused beyond its scope
- **Exploitation steps:**
  1. Fuzz methods (GET, HEAD, POST, PUT, DELETE, PATCH, OPTIONS, TRACE, junk verbs); only use state-changing ones if scope allows. Inspect HEAD Content-Length and TRACE for proxy-added headers.
  2. Try method-override headers such as `X-HTTP-Method-Override: PUT`.
  3. Fuzz origin/identity headers with loopback values, and path-override headers.
  4. Mutate the path with encodings and separators.
  5. Mutate API shape: older version, type juggling, duplicate parameters.
  6. Compare HTTP/1.0 vs 1.1 vs 2, http vs https, direct IP/CNAME vs fronted host, and archive.org history.
  7. Test stale authz: collect session, JWT, refresh token, API key, share link, open websocket as a member, have admin demote/remove them, replay immediately and after 5-15 minutes.
- **Bypasses used:** headers `X-Originating-IP`, `X-Forwarded-For`, `X-Forwarded`, `Forwarded-For`, `X-Remote-IP`, `X-Remote-Addr`, `X-ProxyUser-Ip`, `Client-IP`, `True-Client-IP`, `Cluster-Client-IP` all set to 127.0.0.1; `Host: localhost`, arbitrary Host, or Host removed; `X-Original-URL: /admin/console`, `X-Rewrite-URL: /admin/console`. Paths: `/%2e/path`, `/%252e/path` (double encode), `/%ef%bc%8fpath` (unicode slash), `/SECRET`, `/secret/`, `/secret/.`, `//secret//`, `/./secret/..`, `/;/secret`, `/.;/secret`, `//;//secret`, `/secret.json` (Ruby routing). API: `/v1/` instead of `/v3/`, `{"id":[111]}` or `{"id":{"id":111}}` instead of scalar, JSON/query parameter pollution (`user_id=A&user_id=B`), extra `isAdmin=true`, removed/reordered params, boundary values. Go ServeMux CONNECT skipping path cleaning (use `curl --path-as-is -X CONNECT`). ASP.NET Core `UseStaticFiles()` ahead of `UseAuthorization()` serves files anonymously. Session boolean reused for another object (second-order). Thread-local identity bleed between calls in one batched RPC (DWR). Hop-by-hop headers and request smuggling where authorized.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** a 403 usually protects one parse of the request; vary every dimension (method, header, path form, protocol, host, version) and diff the responses by body bytes, not just status. Also check that revocation actually takes effect.
- **Source:**
  - Title: 403 & 401 Bypasses
  - Author: HackTricks (carlospolop and contributors)
  - URL: https://hacktricks.wiki/en/pentesting-web/403-and-401-bypasses.html
  - Date: unknown (scraped 2026-11-17)
  - Disclosed report: none
  - Raw file: `sources/access-control/02-hacktricks-403-401-bypasses.md`

Tags: taxonomy, reference, 403-bypass, header-bypass, path-normalization, case-encoding, method-based, method-override, session-bypass, privilege-escalation, methodology
