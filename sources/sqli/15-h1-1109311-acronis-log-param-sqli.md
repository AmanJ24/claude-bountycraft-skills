---
title: "SQL injection in https://www.acronis.cz/ via the log parameter"
author: mmg (reporter), Acronis (program)
url: https://hackerone.com/reports/1109311
date_published: 2021-06-11
date_scraped: 2026-11-19
platform: hackerone
disclosed_report: https://hackerone.com/reports/1109311
bounty: Hidden (awarded)
vuln_class: sqli
---

<!-- Raw scraped text below. Public, disclosed report. Severity Medium. Short report. -->

# SQLi in a POST `log` parameter (Acronis), found and confirmed with sqlmap

A SQL injection in `https://www.acronis.cz/` via the POST `log` parameter. Severity Medium; resolved; bounty awarded.

## PoC
Saved the raw request to a file and ran sqlmap against the `log` parameter:
```
sqlmap -p log -r request-cz.txt --current-user --level=2 --risk=2
```
Result: current user `u_acronis@localhost`. (The `-r <file>` form feeds sqlmap a full raw HTTP request, so the injection point is a POST body parameter; `-p log` targets it; `--level/--risk` widen the test set.) The reporter stopped at `--current-user` (minimal, responsible PoC).

## Impact (as stated)
SQLi can bypass the app's authentication and authorization, retrieve the entire database, and potentially execute OS commands for further escalation.

## Lesson
- Any request parameter (here a `log` field in a POST body) can be an injection point, not just obvious ids.
- The clean report pattern: save the raw request to a file, point sqlmap at it with `-r` and `-p <param>`, extract only `--current-user` as proof, and state impact without dumping real data.
- Confirming `current_user` (and whether it is a high-privilege/`root`/`sa`-like account) frames the escalation potential (auth bypass, full dump, OS commands).
