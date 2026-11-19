# Automattic intensedebate: UNION SQLi in a URL path segment (MariaDB)

- **Vuln type / sub-variant:** in-band UNION-based SQLi where the injection point is a URL **path segment**, not a query parameter; MariaDB (MySQL family)
- **Target type:** a reporting endpoint `https://intensedebate.com/commenthistory/$SiteId` that embeds the site id from the path into a query
- **Recon signal:** an id in the URL path that renders data back; adding SQL to the path changes the output
- **Root cause:** the path site-id value is concatenated into a `SELECT` whose results are reflected
- **Exploitation steps:**
  1. Get your own site id (via the dashboard).
  2. Inject a 3-column UNION in the path: `/commenthistory/$SiteId%20union%20select%201,2,@@VERSION%23` (decoded `... union select 1,2,@@VERSION#`).
  3. The reflected column shows `10.1.32-MariaDB`; swap `@@VERSION` for `current_user()` etc.
- **Bypasses used:** `#` to comment the query tail (MySQL/MariaDB); UNION column count matched to the original query; using the reflected column to echo values
- **Impact:** full database access holding private user information (reporter also noted reflected XSS on the same surface); rated Critical (9~10); bounty awarded (hidden)
- **Bounty:** awarded (amount hidden)
- **Lesson / pattern:** test reflected values that live in URL **path** segments, not just `?params`; path-based injection is a common blind spot. A matched-column UNION echoes `@@VERSION`/`current_user()` in whichever column is rendered.
- **Source:**
  - Title: SQL Injection Union Based (intensedebate / Automattic)
  - Author: fuzzme (reporter), Automattic (program)
  - URL: https://hackerone.com/reports/1046084
  - Date: 2021-01-01 (scraped 2026-11-19)
  - Raw file: `sources/sqli/13-h1-1046084-automattic-union-sqli.md`

Tags: union-based, in-band, url-path, mariadb, mysql, dbms-fingerprint, full-db-dump, critical, source-real
