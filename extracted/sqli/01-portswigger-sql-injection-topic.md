# PortSwigger SQLi spine: where it lives in a query, the five detection probes, and the taxonomy

- **Vuln type / sub-variant:** reference taxonomy of SQL injection: logic subversion, UNION, blind (boolean/time/OAST), second-order, and injection in different query parts/contexts
- **Target type:** any app that builds SQL from user input (WHERE is most common, but also UPDATE values, INSERT values, SELECT table/column names, ORDER BY)
- **Recon signal:** submit `'` and look for errors/anomalies; compare base vs modified values; `OR 1=1` vs `OR 1=2`; time-delay payloads; OAST payloads for out-of-band interactions. Any controllable input processed as SQL (query string, JSON, XML body) is a candidate.
- **Root cause:** user input concatenated into a query so it can change the query's structure, not just its data
- **Exploitation steps (the canonical examples):**
  1. Retrieving hidden data: `category=Gifts'--` comments out `AND released=1`; `category=Gifts'+OR+1=1--` returns everything.
  2. Login bypass: username `administrator'--` drops the password check.
  3. UNION: `' UNION SELECT username, password FROM users--` pulls other tables.
  4. Blind: boolean condition / conditional error (divide-by-zero) / time delay / OAST DNS exfiltration.
  5. Examine the DB: version (`SELECT @@version`, Oracle `SELECT * FROM v$version`), tables (`information_schema.tables`).
- **Bypasses used:** SQL comments (`--`, `#`, `/* */`); XML/JSON encoding to dodge keyword filters (e.g. `&#x53;ELECT` for SELECT)
- **Impact:** read other users' data, modify/delete data, auth bypass, escalate to server compromise or DoS; many high-profile breaches
- **Bounty:** reference (not a single finding)
- **Lesson / pattern:** SQLi is possible anywhere input reaches a query's structure, not just the WHERE clause. Detect with the five probes (quote, logic, boolean, time, OAST). Prevent with parameterized queries; for identifiers/ORDER BY (which params can't cover) use allowlists. Never decide case-by-case that data is "trusted".
- **Source:**
  - Title: SQL injection (topic)
  - Author: PortSwigger Web Security Academy
  - URL: https://portswigger.net/web-security/sql-injection
  - Date: reference (scraped 2026-11-19)
  - Raw file: `sources/sqli/01-portswigger-sql-injection-topic.md`

Tags: taxonomy, reference, logic-subversion, auth-bypass-tautology, union-based, blind, second-order, where-clause, order-by, insert, update, json-body, comment-truncation, parameterized-queries, defense
