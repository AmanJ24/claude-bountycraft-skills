# UNION attacks: find the column count, find a string column, then read other tables

- **Vuln type / sub-variant:** in-band UNION-based SQLi (results returned in the response)
- **Target type:** an endpoint that reflects query results (e.g. a product listing / search) where the injected SELECT's output appears
- **Recon signal:** the response echoes query output; a `'` breaks it; you can append `UNION SELECT`
- **Root cause:** input concatenated into a `SELECT` whose results are rendered, so an appended `UNION SELECT` returns attacker-chosen rows
- **Exploitation steps:**
  1. Find the column count. Either `' ORDER BY 1--`, `2--`, `3--` until an "out of range" error, or `' UNION SELECT NULL--`, `NULL,NULL--`, ... until no error (NULL is type-compatible with everything).
  2. Find a string-compatible column: `' UNION SELECT 'a',NULL,NULL,NULL--` then shift `'a'` across columns; an incompatible column errors ("Conversion failed..."), the right one echoes `a`.
  3. Read data: `' UNION SELECT username, password FROM users--`.
  4. If only one column is reflected, concatenate: Oracle `' UNION SELECT username||'~'||password FROM users--` (MySQL `CONCAT()`, MSSQL `+`).
- **Bypasses used:** `NULL` for type compatibility; DB-specific syntax (Oracle needs `FROM dual`; MySQL `--` needs a trailing space or use `#`)
- **Impact:** read arbitrary tables (credentials, PII) in a single request
- **Bounty:** reference (not a single finding)
- **Lesson / pattern:** UNION is the fast path when output is reflected: column count, then a string column, then read. Use information_schema (or guessing) to learn table/column names, and concatenate to fit a single reflected column.
- **Source:**
  - Title: SQL injection UNION attacks
  - Author: PortSwigger Web Security Academy
  - URL: https://portswigger.net/web-security/sql-injection/union-attacks
  - Date: reference (scraped 2026-11-19)
  - Raw file: `sources/sqli/02-portswigger-union-attacks.md`

Tags: union-based, in-band, union-column-count, order-by, union-concat, string-column, information-schema, dbms-fingerprint, reference, methodology
