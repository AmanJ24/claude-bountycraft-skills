---
name: sqli
description: Use when testing whether user input reaches a SQL query and can change its structure: a quote or SQL syntax in a parameter, path segment, JSON/GraphQL field, or header (User-Agent, Referer, Cookie) that causes an error, a different result, or a timing change. Covers recognizing the injection context and the feedback channel, UNION and error-based extraction, blind (boolean/time) and out-of-band (OAST/DNS) exfiltration, second-order SQLi, DBMS fingerprinting, WAF/filter bypass (keyword obfuscation, JSON operators, moving to a less-inspected input), escalation to file read/write and RCE (xp_cmdshell, UDF, plpythonu, Java, INTO OUTFILE), tooling (sqlmap/ghauri), false positives, and reporting. Research/methodology only: it does not send requests to live targets.
---

# SQL Injection

Synthesized from 15 real writeups and references (the PortSwigger SQL injection topic/UNION/blind/cheat-sheet, the YesWeHack bug-bounty guide, Claroty's JSON-operator WAF bypass, a four-DBMS SQLi-to-RCE reference, and disclosed HackerOne/pentest findings: a Critical GraphQL SQLi in a SET SESSION context, a UNION in a URL path, a POST-param auth-bypass, a User-Agent-header blind SQLi, an MSSQL-to-xp_cmdshell RCE at CVSS 10, a second-order OOB-DNS exfiltration, and a WAF bypass that moved the injection to a cookie). Every pattern traces to `sources.md`. Disclosed impact ran from Medium (information disclosure) to Critical (full DB dump, auth bypass, and OS RCE).

**The one idea:** SQL injection is possible wherever user input is concatenated into a query so it can change the query's **structure**, not just its data. To find and exploit it, answer two questions: **(1) where does my input land in the query** (the *context*: a quoted string in `WHERE`, an `ORDER BY`, an `INSERT` tuple, a number, a `SET SESSION`, a header value that gets logged), which decides how to **break out**; and **(2) how does the app reveal the result** (the *channel*: reflected output, a DB error, a timing difference, nothing at all), which decides the **technique** (UNION, error-based, boolean, time, or out-of-band). Break out of the context, read back through the channel, fingerprint the DBMS, then escalate.

**Scope rule:** methodology and guidance only. It does not fire requests at live targets. Real testing needs an authorized, in-scope program and accounts/data you control. SQLi reads and can modify or destroy real data, and `OR 1=1` in a reused input can hit an `UPDATE`/`DELETE`; stay on test data, extract a minimal proof (e.g. `current_user`/version), and never dump real users' data or run OS commands outside an explicitly authorized engagement (see `../../CLAUDE.md`).

**Scope boundary:** this skill is classic SQL injection. NoSQL injection (`{"$ne":...}`, `$where`, `$regex` login bypass) is a separate class, cross-referenced only. ORM-layer injection is a variant of the same root cause. Second-order shares stored-state ground with `business-logic`, but the injection mechanics live here.

## 1. Recognize it

SQLi is the request that is syntactically ordinary but makes the database do something extra because your input became part of the query. The first tells are an error, a changed result set, or a timing shift after unexpected input.

Detection probes, per entry point (from note 01):
- A single quote `'` -> DB error, 500, or a changed/empty response (the strongest first signal).
- Boolean logic: `' OR 1=1--` vs `' OR 1=2--` (or `... AND 1=1` / `AND 1=2`) -> different responses means the condition is being evaluated.
- A base-vs-different value that should and should not match, and compare responses.
- A time-delay payload (DBMS-specific SLEEP/WAITFOR/pg_sleep) -> a response that slows by the injected amount.
- An OAST payload that triggers an out-of-band DNS/HTTP interaction you monitor.
- A verbose DB error that quotes the query (e.g. "Unclosed quotation mark...", "PG::SyntaxError") confirms it and maps the context.

## 2. Where it hides

Most SQLi is in a `WHERE` clause, but it occurs anywhere input reaches a query. Full catalog in `references/detection-and-contexts.md`.

- **Query parameters and search fields** (the classic `?id=`, `?category=`, `?s=`): WHERE-clause string or numeric context (notes 06, 24).
- **URL path segments** (`/commenthistory/<id>`): easy to overlook, same as a param (note 13).
- **POST body fields**, including non-obvious ones like a `log` parameter (note 15).
- **JSON and GraphQL** fields/variables: a GraphQL *parameter* (not an input field) in a `SET SESSION` context, or a JSON `text` field (notes 12, 19).
- **HTTP headers** the app logs or queries: `User-Agent`, `Referer`, `X-Forwarded-For`, `Cookie` (note 18); sqlmap needs higher `--level` to reach these.
- **Other query clauses**: `ORDER BY`, `INSERT`/`UPDATE` values, and table/column-name positions (notes 01, 06) where parameterization cannot help.
- **Stored fields reused later** (username, preference, report param): second-order SQLi (notes 06, 21).

## 3. Test methodology

Find the context, break out, pick the channel, fingerprint, escalate, prove minimally. Full step-by-step in `references/detection-and-contexts.md`.

1. **Probe every entry point** with the six signals above. A `'` that errors or changes the response, or a boolean/time difference, is your lead.
2. **Determine the context** from the error or behaviour: quoted string vs number, which clause (`WHERE`/`ORDER BY`/`INSERT`/`SET SESSION`), how many columns. The context dictates the breakout (quote, parenthesis, comment, or a tuple-closing payload). See note 06.
3. **Pick the channel by what the app reveals:**
   - Output reflected -> **UNION** (`references/in-band-union-and-error.md`): find the column count (`ORDER BY n` / `UNION SELECT NULL,...`), find a string column, read other tables.
   - A catchable DB error -> **error-based** (`CAST`/`EXTRACTVALUE`/conditional divide-by-zero).
   - A visible true/false difference -> **boolean-blind**: extract one char at a time with `SUBSTRING` + binary search.
   - Graceful errors, no difference -> **time-based**: conditional SLEEP/WAITFOR.
   - Async / nothing -> **out-of-band (OAST)**: DNS exfiltration (prefer it anyway; fast and direct). See `references/blind-oob-and-second-order.md`.
4. **Fingerprint the DBMS** (version/error/time method that works identifies the engine) and pull its exact syntax from `references/dbms-specific-and-escalation.md`.
5. **Escalate** where authorized: enumerate via information_schema, then consider file read/write and RCE (xp_cmdshell, UDF, plpythonu, Java stored proc, INTO OUTFILE) if the DB account is privileged.
6. **If a WAF blocks you**, obfuscate keywords, use JSON/dialect syntax, or move the injection to a less-inspected input (cookie/header). See `references/waf-bypass-tooling-and-reporting.md`.
7. **Automate carefully** with sqlmap/ghauri once you understand the context; confirm manually first.
8. **Stop at proof** on test data: a `current_user`, version, or one controlled record, not a full dump of real data, and no OS commands outside an authorized engagement.

## 4. Known bypass / evasion patterns

Summaries; full arsenals in `references/waf-bypass-tooling-and-reporting.md` and `references/dbms-specific-and-escalation.md`.

- **Keyword/filter obfuscation (note 06):** case variation (`SeLeCT`), inline comments for whitespace (`SELECT/**/FROM`), newlines (`%0A`), parentheses (`OR(1=1)`), MySQL versioned comments (`/*!SELECT*/`), and encoding (URL/`CHAR()`/XML entity like `&#x53;ELECT`).
- **JSON-operator WAF bypass (note 07):** append a valid-but-unmodelled JSON operator (PostgreSQL `@>`, `->`, `?`) so the WAF's SQL parser fails while the DB still executes; beat Palo Alto, AWS, Cloudflare, F5, Imperva.
- **Move to a less-inspected input (note 22):** if the GET param is WAF-guarded, deliver the same value via a cookie or header (ParamMiner finds hidden ones); WAFs often inspect inputs unevenly.
- **Context breakout (note 06):** a classic `x' OR 1=1 -- -` fails inside an `INSERT` tuple; use `x'||'y` or close the tuple `x', (SELECT ...), 'y')-- -`.
- **Scanner blind spots (note 12):** non-WHERE contexts (`SET SESSION`) make automated scanners see 500s and miss the bug; test manually and watch for syntax errors.

## 5. Common false positives

- **A slow response that is network noise, not a SLEEP.** Require a **linearly scaling** delay across multiple sleep values (5s/10s/15s), not one slow hit (note 24).
- **A reflected value that was not evaluated.** The input appearing in the response is not injection; a `'` must change behaviour or error, and a boolean must flip the result.
- **A WAF's uniform empty/blocked response.** An empty `200, Content-Length: 0` on every payload is a WAF reacting, not "not injectable"; confirm the base behaviour manually before concluding (note 22).
- **A generic error page vs a DB error.** Only a database error (or a provable boolean/time/OAST difference) confirms SQLi; a 500 alone may be input validation.
- **An `OR 1=1` that "worked" but is dangerous.** It can reach an `UPDATE`/`DELETE` and change data; confirm with a safe boolean (`AND 1=1` vs `AND 1=2`) instead of a blanket `OR 1=1`.

## 6. Reporting

Lead with the injectable input, the context, and a minimal, non-destructive proof. Full guidance in `references/waf-bypass-tooling-and-reporting.md`.

- **Name the input and context:** "the `s` GET parameter is injected into a MySQL `WHERE` clause", "the User-Agent header is logged into a query". Give the exact request.
- **Prove it minimally:** a scaling time delay, a boolean true/false pair, an OAST DNS hit, or `current_user`/`@@version` extracted. For a responsible PoC, `sqlmap -r <request> -p <param> --current-user` and stop (notes 15, 24). Do not dump real users' data.
- **State impact concretely:** data read (and whose), auth bypass, data modification, and the DBMS + the account's privilege (a `sa`/`root`/`FILE`-privileged account means file read/write or RCE is reachable). Map Medium (info disclosure) up to Critical (full dump / auth bypass / OS RCE).
- **Remediation:** parameterized queries / prepared statements (primary); least-privilege DB account; suppress verbose errors; disable and lock down `xp_cmdshell`; WAF only as a secondary layer.

## 7. Sources

See `sources.md` for the full list (title, author, URL, disclosed-report link, bounty) behind every claim above, keyed to the extracted notes in `extracted/sqli/`. One member-only writeup (InfoSec boolean-blind bootcamp) is pending retrieval; the boolean technique it covers is already sourced from notes 03/18/22/24.
