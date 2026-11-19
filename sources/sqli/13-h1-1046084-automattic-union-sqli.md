---
title: "SQL Injection Union Based (intensedebate / Automattic)"
author: fuzzme (reporter), Automattic (program)
url: https://hackerone.com/reports/1046084
date_published: 2021-01-01
date_scraped: 2026-11-19
platform: hackerone
disclosed_report: https://hackerone.com/reports/1046084
bounty: Hidden (awarded)
vuln_class: sqli
---

<!-- Raw scraped text below. Public, disclosed report. Severity Critical (9~10). MariaDB. -->

# UNION-based SQLi in a URL path segment (intensedebate, MariaDB)

Automattic's intensedebate.com. The site id in the URL path `https://intensedebate.com/commenthistory/$YourSiteId` was injected directly into a SQL query. Severity Critical (9~10), bounty awarded (hidden).

## Steps to reproduce
1. Log in to intensedebate.com, create a site (/install), find your site id via /user-dashboard -> /dash/$SiteId.
2. Visit the vulnerable endpoint with your site id: `/commenthistory/$YourSiteId`.
3. Trigger UNION SQLi (note the injection is in the PATH, not a query parameter):
```
https://intensedebate.com/commenthistory/$YourSiteId%20union%20select%201,2,@@VERSION%23
```
Decoded: `$YourSiteId union select 1,2,@@VERSION#`. The page then renders `10.1.32-MariaDB` (the version appears in the reflected output column).
4. Swap `@@VERSION` for `current_user()` etc. to enumerate further.

## Key points
- Injection point is a **URL path segment** (`/commenthistory/<id>`), not a `?param=`; path parameters are easy to overlook.
- Classic 3-column UNION: `union select 1,2,@@VERSION#`. The reflected column (3rd) shows the injected value. `#` comments out the query tail (MySQL/MariaDB).
- Backend is MariaDB (MySQL-family), so `#` and `@@VERSION`/`current_user()` apply.

## Impact
Full database access holding private user information (and the reporter also noted reflected XSS on the same surface). Resolved and patched.

## Lesson
Test every reflected value, including id segments embedded in the URL path, with a UNION whose column count matches the original query; the version/`current_user()` appears in whichever column is echoed back. Path-based injection is a common blind spot.
