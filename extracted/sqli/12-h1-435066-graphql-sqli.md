# HackerOne GraphQL SQLi: a GraphQL parameter in a SET SESSION context that scanners miss

- **Vuln type / sub-variant:** in-band/stacked-query SQLi via a GraphQL parameter, in a non-WHERE context (`SET SESSION ... TO '<injection>'`), PostgreSQL
- **Target type:** a GraphQL endpoint (`/graphql`) that passes a GraphQL parameter (`embedded_submission_form_uuid`), not an input field, into a raw query
- **Recon signal:** a stray `PG::SyntaxError` in the backend; a GraphQL parameter that reaches SQL unsanitized (input fields were sanitized, parameters were not)
- **Root cause:** when converting the GraphQL query to SQL, a sanitization step was skipped; the parameter was concatenated into `safe_query += "SET SESSION #{key} TO #{value};"` and executed
- **Exploitation steps:**
  1. Inject via the parameter (URL or body, JSON or form-encoded; Rails `params` accepts any): `?embedded_submission_form_uuid=1';SELECT 1;SELECT pg_sleep(30);--'`.
  2. Confirm with timing: `pg_sleep(5)` -> 5.7s, `pg_sleep(10)` -> 10.6s, `pg_sleep(30)` -> 500 (backend aborts at 28s).
- **Bypasses used:** stacked queries (`;SELECT ...`) in PostgreSQL; the unusual `SET SESSION` context, which caused automated scanners (expecting a WHERE clause) to see 500s and not report it
- **Impact:** executed in the per-user "secure" schema, but a schema switch would expose the full DB; rated Critical (10.0). Not exploited (confirmed by log analysis). No bounty (internal).
- **Bounty:** none (HackerOne internal report)
- **Lesson / pattern:** GraphQL variables/parameters are user input and need parameterization like any other. Non-WHERE contexts (here `SET SESSION`) evade scanners, so watch for DB syntax errors and test manually. Per-user derived views limit blast radius but are not a fix.
- **Source:**
  - Title: SQL injection in GraphQL endpoint through embedded_submission_form_uuid parameter
  - Author: jobert (reporter), HackerOne (program)
  - URL: https://hackerone.com/reports/435066
  - Date: 2018-11-30 (scraped 2026-11-19)
  - Raw file: `sources/sqli/12-h1-435066-graphql-sqli.md`

Tags: graphql, set-session, stacked-queries, time-based, postgresql, pg-sleep, scanner-blind-spot, critical, information-disclosure, source-real
