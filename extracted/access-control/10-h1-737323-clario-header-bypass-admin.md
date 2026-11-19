# Clario: nginx 403 on /admin bypassed via X-Rewrite-Url (source is partial)

- **Vuln type / sub-variant:** URL-based access control bypass via request-URL override header
- **Target type:** account portal behind a front nginx proxy
- **Recon signal:** /admin/login returns 403 from the front server, suggesting a path-based block at the proxy while the backend app is behind it
- **Root cause:** the front server blocks by request path but the backend honors X-Rewrite-Url / X-Original-URL and routes on that instead; the proxy does not strip or check these headers
- **Exploitation steps:** (source is partial; only the program summary is public)
  1. Request GET /admin/login directly and observe 403.
  2. Send GET / with header `X-Rewrite-Url: admin/login` (the `X-Original-URL` variant is the same idea).
  3. The backend serves the admin login page.
- **Bypasses used:** `X-Rewrite-Url: admin/login` (note: no leading slash worked) on request to `/`; `X-Original-URL` equivalent
- **Impact:** access to forbidden admin files/directories (login page shown); Medium (6.1) (source is partial)
- **Bounty:** $300
- **Lesson / pattern:** whenever a 403 comes from a proxy, retry with path-override headers against `/`; a 200 with different content proves the backend processes the header.
- **Source:**
  - Title: Bypass front server restrictions and access to forbidden files and directories through X-Rewrite-Url/X-original-url header on account.mackeeper.com
  - Author: rumiljonov
  - URL: https://hackerone.com/reports/737323
  - Date: 2019-11-14 (disclosed 2020-05-15)
  - Disclosed report: https://hackerone.com/reports/737323
  - Raw file: `sources/access-control/10-h1-737323-clario-header-bypass-admin.md`

Tags: url-based, header-bypass, 403-bypass, unprotected-admin, admin-access
