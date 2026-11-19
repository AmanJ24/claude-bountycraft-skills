---
title: "{JS-ON: Security-OFF}: Abusing JSON-Based SQL to Bypass WAF"
author: Noam Moshe (Claroty Team82)
url: https://claroty.com/team82/research/js-on-security-off-abusing-json-based-sql-to-bypass-waf
date_published: 2022-12-08
date_scraped: 2026-11-19
platform: other
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public research. Condensed from the full article (images omitted). -->

# Abusing JSON-based SQL to bypass WAFs (Team82)

## Executive summary
- A generic bypass of industry-leading WAFs: append/prepend JSON syntax to a SQL injection payload that the WAF cannot parse.
- Major WAF vendors lacked JSON-in-SQL support, though most database engines have supported it for ~a decade.
- WAFs easily detect normal SQLi, but JSON syntax inside the SQL left them blind.
- Confirmed against five vendors: Palo Alto Networks, AWS, Cloudflare, F5, Imperva (all notified and fixed).
- Impact: bypass the WAF, then use an existing SQLi to exfiltrate data.

## The core mismatch
WAFs detect SQLi by recognizing SQL syntax. JSON operators are valid SQL in modern engines (PostgreSQL, MySQL, MSSQL, SQLite support JSON) but are "less prevalent" syntax the WAF parsers did not understand. When a payload uses JSON operators, the WAF fails to flag it while the database still executes it.

## The technique
The researchers found their SQLi (over the cloud, a PostgreSQL backend). A normal SQLi payload was blocked by AWS WAF. Prepending/including JSON syntax defeated it. The key operator used was PostgreSQL's JSONB containment operator `@>` (checks whether the right JSON is contained in the left):
- Inserting a JSON operator such as `@>` into the payload "threw the WAF into a loop" (its SQL parser could not handle it), so the request passed uninspected.
- With the WAF bypassed, they ran their UNION-based SQLi. In PostgreSQL the UNION must return matching types; they used `CAST` and PostgreSQL row expansion to return the 8-byte values the original query expected.

PostgreSQL JSON operators useful for this class: `@>`, `<@` (containment), `->`, `->>` (field access), `#>`, `?`, `?|`, `?&`. Example shape of a bypass payload:
```
' OR '1'='1'@>'{"x":"y"}  -- (JSON operator appended so the WAF's SQL parser chokes, DB still evaluates)
```
(Exact payloads vary by injection context and engine; the principle is: wrap or suffix the SQL with a valid-but-unusual JSON operator the WAF does not model.)

## Generalization
The root issue is non-conformance between database engines and SQLi-detection solutions: JSON-in-SQL is a real feature the DB supports but the WAF does not inspect. Any WAF that does not model the full SQL dialect of the backend can be bypassed by using syntax it does not recognize (here JSON; the same idea applies to other under-modeled syntax).

## Defense
WAF vendors added JSON parsing to their SQLi inspection. Defenders should not rely on a WAF as the primary control; use parameterized queries. The lesson for testers: when a WAF blocks a SQLi, try JSON operators and other dialect-specific syntax the WAF may not understand.
