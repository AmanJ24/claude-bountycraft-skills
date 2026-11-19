# JSON-based SQL beats WAFs: use valid-but-unmodelled JSON operators the WAF's parser can't handle

- **Vuln type / sub-variant:** WAF bypass for SQLi using JSON syntax/operators that the WAF's SQL parser does not understand (novel generic bypass)
- **Target type:** apps behind an industry WAF with a backend DB that supports JSON-in-SQL (PostgreSQL, MySQL, MSSQL, SQLite); demonstrated over a PostgreSQL cloud target
- **Recon signal:** a known SQLi is blocked by the WAF; the backend engine supports JSON operators
- **Root cause:** non-conformance between the WAF and the DB: the WAF detects SQLi by recognizing SQL syntax, but JSON-in-SQL is a real feature the DB executes while the WAF's parser cannot model it, so a JSON-laced payload passes uninspected
- **Exploitation steps:**
  1. Confirm the SQLi and that a plain payload is WAF-blocked.
  2. Insert a valid JSON operator into the payload. The researchers used PostgreSQL's JSONB containment operator `@>` (checks if the right JSON is contained in the left); it "threw the WAF into a loop" so the request passed.
  3. With the WAF bypassed, run the actual injection (UNION-based), using `CAST`/row expansion to return the data types the original query expected.
- **Bypasses used:** PostgreSQL JSON operators (`@>`, `<@`, `->`, `->>`, `#>`, `?`, `?|`, `?&`) appended/wrapped into the payload; generalizes to any dialect syntax the WAF under-models
- **Impact:** defeat the WAF and exfiltrate data via an existing SQLi; worked against Palo Alto, AWS, Cloudflare, F5, Imperva (all later fixed)
- **Bounty:** reference (vendor research)
- **Lesson / pattern:** a WAF only blocks the SQL dialect it models. When a SQLi is WAF-blocked, try JSON operators and other backend-specific syntax the WAF may not parse. Do not rely on a WAF as the control; parameterize.
- **Source:**
  - Title: {JS-ON: Security-OFF}: Abusing JSON-Based SQL to Bypass WAF
  - Author: Noam Moshe (Claroty Team82)
  - URL: https://claroty.com/team82/research/js-on-security-off-abusing-json-based-sql-to-bypass-waf
  - Date: 2022-12-08 (scraped 2026-11-19)
  - Raw file: `sources/sqli/07-claroty-json-based-sql-waf-bypass.md`

Tags: waf-bypass, json-operator, postgresql, union-based, dialect-syntax, research, reference, defense
