---
title: "SQL injection in GraphQL endpoint through embedded_submission_form_uuid parameter"
author: jobert (reporter), HackerOne (program)
url: https://hackerone.com/reports/435066
date_published: 2018-11-30
date_scraped: 2026-11-19
platform: hackerone
disclosed_report: https://hackerone.com/reports/435066
bounty: None (internal report)
vuln_class: sqli
---

<!-- Raw scraped text below. Public, disclosed report. Severity Critical (10.0). -->

# SQLi in HackerOne's /graphql endpoint (embedded_submission_form_uuid), PostgreSQL

Reported by HackerOne staff after @tomdev noticed a `PG::SyntaxError` in the backend. Severity Critical (10.0). Not exploited (confirmed via log analysis). No bounty (internal).

## Root cause
The `/graphql` endpoint's `embedded_submission_form_uuid` GraphQL **parameter** (not an input field) was not sanitized. Input fields are sanitized; GraphQL parameters were not designed to take raw user input. The vulnerable Ruby code built a raw query from parameters:
```ruby
new_parameters.each do |key, value|
  safe_query += "SET SESSION #{key} TO #{value};"   # user value concatenated
end
connection.query(safe_query)
```
HackerOne uses two schemas: `public` (all data) and `secure` (per-user derived views, default for GraphQL, limits IDOR/SQLi exposure). The injection ran in the `secure` schema context, but switching schemas would raise impact, hence Critical on confidentiality.

## The injection context matters (key lesson)
The injection point was NOT a `WHERE` clause but inside `SET SESSION <parameter> TO '<injection>'`. Most automated scanners assume a WHERE-clause context, so they saw 500 errors and did not report the SQLi. A human found it from a stray `PG::SyntaxError`.

## PoC (stacked-query time-based, PostgreSQL)
```
curl -X POST "https://hackerone.com/graphql?embedded_submission_form_uuid=1%27%3BSELECT%201%3BSELECT%20pg_sleep(30)%3B--%27"
```
Decoded: `1';SELECT 1;SELECT pg_sleep(30);--'`. Timing proof:
```
pg_sleep(5)  -> 5.726s total
pg_sleep(1)  -> 1.631s total
pg_sleep(10) -> 10.557s total
pg_sleep(30) -> 500 (backend aborts at 28s)
```

## Detection / response (defender side)
Identifiers used to hunt for abuse in logs: POST to `/graphql`, the `embedded_submission_form_uuid` param (URL or body, JSON or form-encoded via Rails `params`), and a single quote in the input. nginx logs lacked body data; Rails logs had it but were hard to query. Only their own `pg_sleep` payloads from trusted IPs succeeded. Takeaway: build a queryable JSON log pipeline capturing full request context.

## Lesson
A GraphQL variable/parameter is user input and must be parameterized like any other. Non-WHERE injection contexts (here `SET SESSION`) evade scanners, so manual testing and watching for DB syntax errors matter. Per-user "secure" views limit blast radius but are not a substitute for parameterization.
