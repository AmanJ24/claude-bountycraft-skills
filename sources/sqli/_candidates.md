# SQL Injection: candidate writeups (Find + Filter)

Find/Filter done 2026-11-19. Scope: classic SQL injection end to end. Sub-variants to cover:
- In-band: error-based, UNION-based (column count, string column, multi-value concat).
- Blind: boolean-based (conditional responses), time-based (SLEEP/BENCHMARK/WAITFOR/pg_sleep).
- Out-of-band (OOB): DNS/HTTP exfil via EXTRACTVALUE/xmltype, xp_dirtree, LOAD_FILE, UNC, Collaborator.
- Second-order (stored then triggered later).
- Contexts: WHERE, ORDER BY, INSERT/UPDATE, LIMIT, headers (User-Agent, Referer, X-Forwarded-For, Cookie), JSON/REST, GraphQL.
- Auth bypass (tautology / login bypass).
- WAF bypass / filter evasion (incl. JSON-operator bypass, comment/whitespace tricks, encoding).
- DBMS-specific escalation: file read/write (INTO OUTFILE, LOAD_FILE), RCE (xp_cmdshell, COPY TO PROGRAM, stacked queries), DBMS fingerprinting (MySQL/PostgreSQL/MSSQL/Oracle/SQLite).
- Tooling: sqlmap, ghauri.

Scope boundary: NoSQL injection (Mongo `$ne`/`$regex`, etc.) is CROSS-REFERENCED and deferred to a future `nosqli` skill, not covered in depth here. ORM-layer injection noted as a variant. Second-order shares ground with `business-logic` (stored state) but the injection mechanics live here.

Target: 15 to 25 kept. (24 selected.) Ask before scraping more than ~25 pages in one run.

Status: candidate | keep | drop | scraped

## References / methodology (reliably public)

| #  | Title | Author | URL | Sub-variant | Status |
|----|-------|--------|-----|-------------|--------|
| 01 | SQL injection (topic + labs, the spine) | PortSwigger | https://portswigger.net/web-security/sql-injection | taxonomy, logic subversion, examining DB | candidate |
| 02 | SQL injection UNION attacks | PortSwigger | https://portswigger.net/web-security/sql-injection/union-attacks | in-band UNION: column count, string col, concat | candidate |
| 03 | Blind SQL injection | PortSwigger | https://portswigger.net/web-security/sql-injection/blind | boolean, time, error, OOB exfil | candidate |
| 04 | SQL injection cheat sheet | PortSwigger | https://portswigger.net/web-security/sql-injection/cheat-sheet | DBMS-specific syntax reference | candidate |
| 05 | SQL injection (pentest guide) | HackTricks | https://book.hacktricks.wiki/en/pentesting-web/sql-injection/index.html | end-to-end detection + DBMS arsenal | candidate |
| 06 | SQLi: the ultimate Bug Bounty guide | YesWeHack | https://www.yeswehack.com/learn-bug-bounty/vulnerability-vectors-sql-injection | bug-bounty methodology, all variants | candidate |
| 07 | JSON-based SQL to bypass WAF (Team82) | Claroty | https://claroty.com/team82/research/js-on-security-off-abusing-json-based-sql-to-bypass-waf | WAF bypass via JSON operators (novel) | candidate |
| 08 | SQL Injection Bypassing WAF | OWASP | https://community.owasp.org/attacks/SQL_Injection_Bypassing_WAF | filter/WAF evasion techniques | candidate |
| 09 | SQL Injection Cheat Sheet (DBMS, error-based) | Invicti | https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet | DBMS stored procs, error-based, file ops | candidate |
| 10 | Out-of-Band SQL Injection | Invicti | https://www.invicti.com/learn/out-of-band-sql-injection-oob-sqli | OOB channels, when/why | candidate |

## Disclosed real findings

| #  | Title | Author / Program | URL | Sub-variant | Status |
|----|-------|------------------|-----|-------------|--------|
| 11 | www.drivegrab.com SQLi (Formidable Pro, $4,500) | jouko / Grab | https://hackerone.com/reports/273946 | plugin SQLi -> critical, in the wild | candidate |
| 12 | SQLi in GraphQL endpoint (embedded_submission_form_uuid) | jobert / HackerOne | https://hackerone.com/reports/435066 | stacked-query time-based in GraphQL | candidate |
| 13 | SQL Injection Union Based (intensedebate) | reporter / Automattic | https://hackerone.com/reports/1046084 | UNION-based in URL path | candidate |
| 14 | SQLi through /include/findusers (boolean) | reporter / ImpressCMS | https://hackerone.com/reports/1081145 | boolean-based blind | candidate |
| 15 | SQL injection (auth/authz bypass, full DB) | reporter / Acronis | https://hackerone.com/reports/1109311 | SQLi -> auth bypass + dump | candidate |
| 16 | SQLi on admin.acronis.host (dev service) | reporter / Acronis | https://hackerone.com/reports/923020 | real-world SQLi, scope nuance | candidate |
| 17 | Blind SQL Injection (DoD) | reporter / U.S. DoD | https://hackerone.com/reports/2597543 | OR-based blind, `-4963 OR 2121=2121` | candidate |
| 18 | Boolean Blind SQLi via User-Agent (.mil) | reporter / U.S. DoD | https://hackerone.com/reports/2599826 | injection in HTTP header context | candidate |

## Technique / walkthrough writeups

| #  | Title | Author | URL | Sub-variant | Status |
|----|-------|--------|-----|-------------|--------|
| 19 | From MS SQL Injection to RCE in a Production CRM | velstadt | https://velstadt.com/ms-sql-injection-to-rce-production-crm/ | MSSQL -> xp_cmdshell RCE, sqlmap | candidate |
| 20 | Exploiting SQLi for RCE: Four Techniques Across DBs | 0x3adly | https://medium.com/@0x3adly/from-sql-injection-to-rce-leveraging-vulnerability-for-maximum-impact-2fb356907eed | RCE across MySQL/MSSQL/PostgreSQL/Oracle | candidate |
| 21 | Second Order SQLi with Stored Procedures (DNS OOB) | NetSPI | https://www.netspi.com/blog/technical-blog/web-application-pentesting/second-order-sql-injection-with-stored-procedures-dns-based-egress/ | second-order + OOB DNS egress | candidate |
| 22 | Exploiting an SQL injection with WAF bypass | Vaadata | https://www.vaadata.com/en/blog/exploiting-an-sql-injection-with-waf-bypass/ | real pentest WAF-bypass walkthrough | candidate |
| 23 | Boolean-Based Blind SQLi (Bootcamp #28) | InfoSec Writeups | https://infosecwriteups.com/bug-bounty-bootcamp-28-boolean-based-blind-sql-injection-extracting-data-one-true-false-at-a-abe69e3de702 | boolean exfil one bit at a time | candidate |
| 24 | Time-Based Blind SQLi: confirm + exploit with curl/ghauri | boumriga | https://medium.com/@boumriga.bahmed/time-based-blind-sql-injection-confirming-and-exploiting-with-curl-and-ghauri-3c7bf94aaffd | time-based, tooling (ghauri) | candidate |

Notes:
- Expect some HackerOne reports to hit the sign-in / limited-disclosure wall (business-logic lost 5; access-control lost 0). References 01-10 are reliably public; the Medium/vendor writeups (19-24) are public.
- PayloadsAllTheThings SQLi is a strong payload backfill if DBMS coverage is thin after extraction: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
- NoSQL injection deliberately deferred to a future skill; this skill cross-references it for the `{"$ne":...}` login-bypass class.
- Backlog if gaps surface: PortSwigger "Hidden" research (SQLi in ORM/second-order), sqlmap official wiki for automation depth, bugbountyexplained "128 reports: SQLi to RCE/file read" (paid, skip).
