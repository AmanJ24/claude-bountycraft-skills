# Acronis: SQLi in a POST `log` parameter, confirmed minimally with sqlmap -r

- **Vuln type / sub-variant:** SQLi in a POST body parameter (`log`), a non-obvious field; auth/authz-bypass class impact
- **Target type:** a marketing/support site (www.acronis.cz) accepting a POST with a `log` parameter
- **Recon signal:** an uncommon POST parameter (`log`) that reaches a query; a `'` or sqlmap heuristic flags it
- **Root cause:** the `log` value concatenated into a backend query
- **Exploitation steps:**
  1. Save the raw HTTP request to a file.
  2. Run `sqlmap -p log -r request-cz.txt --current-user --level=2 --risk=2`.
  3. Recover `current_user` = `u_acronis@localhost`; stop there (minimal PoC).
- **Bypasses used:** none needed; `-r <file>` feeds sqlmap the full request so it injects the POST param; `--level/--risk` widen tests
- **Impact:** stated as auth/authz bypass, full DB retrieval, and potential OS command execution; rated Medium; bounty awarded
- **Bounty:** awarded (hidden)
- **Lesson / pattern:** any request field, including a plain POST `log` parameter, can be injectable. The clean responsible-PoC pattern: `sqlmap -r <raw request> -p <param> --current-user` only, no data dump, then state impact. Checking `current_user`/privilege level frames the escalation (auth bypass, dump, OS commands).
- **Source:**
  - Title: SQL injection in https://www.acronis.cz/ via the log parameter
  - Author: mmg (reporter), Acronis (program)
  - URL: https://hackerone.com/reports/1109311
  - Date: 2021-06-11 (scraped 2026-11-19)
  - Raw file: `sources/sqli/15-h1-1109311-acronis-log-param-sqli.md`

Tags: post-param, sqlmap, current-user, auth-bypass, responsible-poc, medium, source-real
