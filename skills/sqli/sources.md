# Sources: SQL Injection skill

16 sources distilled into `extracted/sqli/`, synthesized into `SKILL.md` and the five `references/` files. Raw scrapes in `sources/sqli/`. Paraphrased, never copied; only citation lines overlap source text. Source #23 (member-only Medium) was supplied by the user as a legitimate download in `Manual_downloads/` and folded in citing the original URL.

Scope: classic SQL injection end to end (in-band error/UNION, blind boolean/time, OOB/OAST, second-order, injection contexts, WAF bypass, DBMS-specific escalation to file-read/RCE, tooling). NoSQL injection is a separate class, cross-referenced only. ORM-layer injection is a variant of the same root cause.

## References and methodology (no single finding)

| # | Title | Author | URL | Role |
|---|-------|--------|-----|------|
| 01 | SQL injection (topic) | PortSwigger | https://portswigger.net/web-security/sql-injection | spine: taxonomy, detection, contexts, prevention |
| 02 | SQL injection UNION attacks | PortSwigger | https://portswigger.net/web-security/sql-injection/union-attacks | UNION column count, string column, concat |
| 03 | Blind SQL injection | PortSwigger | https://portswigger.net/web-security/sql-injection/blind | boolean, error, time, OAST |
| 04 | SQL injection cheat sheet | PortSwigger | https://portswigger.net/web-security/sql-injection/cheat-sheet | DBMS-specific syntax arsenal |
| 06 | SQLi: the ultimate Bug Bounty guide | Alex Brumen (YesWeHack) | https://www.yeswehack.com/learn-bug-bounty/vulnerability-vectors-sql-injection | bug-bounty methodology, contexts, breakout, WAF obfuscation |
| 07 | {JS-ON: Security-OFF}: Abusing JSON-Based SQL to Bypass WAF | Noam Moshe (Claroty Team82) | https://claroty.com/team82/research/js-on-security-off-abusing-json-based-sql-to-bypass-waf | JSON-operator WAF bypass (novel) |
| 20 | Exploiting SQL Injection for RCE: Four Techniques Across Popular Databases | Anas Eladly (0x3adly) | https://medium.com/@0x3adly/from-sql-injection-to-rce-leveraging-vulnerability-for-maximum-impact-2fb356907eed | SQLi->RCE per DBMS (xp_cmdshell/UDF/plpythonu/Java) |

## Disclosed / real-world findings

| # | Title | Author / Program | URL | Disclosed report | Bounty / Severity |
|---|-------|------------------|-----|------------------|-------------------|
| 12 | SQLi in GraphQL endpoint (embedded_submission_form_uuid) | jobert / HackerOne | https://hackerone.com/reports/435066 | yes | none (internal); Critical 10.0 |
| 13 | SQL Injection Union Based (intensedebate) | fuzzme / Automattic | https://hackerone.com/reports/1046084 | yes | awarded (hidden); Critical |
| 15 | SQL injection via the log parameter (acronis.cz) | mmg / Acronis | https://hackerone.com/reports/1109311 | yes | awarded (hidden); Medium |
| 18 | Boolean Blind SQLi via User-Agent (.mil) | iamunixtz / U.S. DoD | https://hackerone.com/reports/2599826 | yes | VDP; Medium |
| 19 | From MS SQL Injection to RCE in a Production CRM | Velstadt | https://velstadt.com/ms-sql-injection-to-rce-production-crm/ | none (pentest) | CVSS 10.0 |
| 21 | Second Order SQLi with Stored Procedures (DNS OOB) | Deepak Dhasmana (NetSPI) | https://www.netspi.com/blog/technical-blog/web-application-pentesting/second-order-sql-injection-with-stored-procedures-dns-based-egress/ | none (pentest) | not stated |
| 22 | Exploiting an SQL injection with WAF bypass | Benoit Philippe (Vaadata) | https://www.vaadata.com/en/blog/exploiting-an-sql-injection-with-waf-bypass/ | none (pentest) | not stated |
| 23 | Bug Bounty Bootcamp #28: Boolean-Based Blind SQL Injection | Aman Sharma (InfoSec Write-ups) | https://infosecwriteups.com/bug-bounty-bootcamp-28-boolean-based-blind-sql-injection-extracting-data-one-true-false-at-a-abe69e3de702 | none (member-only; user-supplied) | not stated |
| 24 | Time-Based Blind SQLi: confirm + exploit with curl and Ghauri | Bahmed Boumriga | https://medium.com/@boumriga.bahmed/time-based-blind-sql-injection-confirming-and-exploiting-with-curl-and-ghauri-3c7bf94aaffd | none (active H1 program, redacted) | not stated; Critical |

## Candidates filtered out / deferred (in `sources/sqli/_candidates.md`, not scraped)
- 05 HackTricks SQLi, 08 OWASP WAF-bypass, 09 Invicti cheat-sheet, 10 Invicti OOB: DBMS/OOB/WAF reference content already covered by 04 + 06 + 07.
- 11 Grab/Formidable, 14 ImpressCMS boolean, 16 Acronis #923020, 17 DoD #2597543 blind: additional real cases redundant with the error-based/boolean/auth-bypass cases already kept (12/13/15/18/19/22/24).
- Payload backfill if a gap surfaces: PayloadsAllTheThings SQL Injection (https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection).

## Notes on provenance
- Member-only article (23): the user downloaded the full InfoSec Write-ups article legitimately to `Manual_downloads/` (project paywall rule: never bypass, the user retrieves it); folded in citing the original URL, not the freedium mirror.
- Cross-skill: OOB/DNS exfiltration shares tooling with the `ssrf` skill (Collaborator/Interactsh); second-order shares stored-state ground with `business-logic`; NoSQLi is a future separate skill.
- Partial/redacted: 18 and 24 have redacted targets (DoD / active program); 19, 21, 22 are anonymized pentests. Techniques are preserved; target identities are not claimed.
