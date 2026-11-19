# case-03: UNION attack, multiple values in a single column (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column
- **Skill path:** SKILL.md section 3 (UNION concat); references/in-band-union-and-error.md; note 02.
- **Date:** 2026-11-19, agent-run live.

## Steps
1. Original query returns 2 columns; only the 2nd is string-compatible.
2. `' UNION SELECT NULL, username||'~'||password FROM users--` (PostgreSQL `||` concatenation with a `~` separator) returned `administrator~<password>` in the single reflected column.
3. Logged in as administrator.

## Result
**Green banner: "LAB Solved"**. Validated single-column concatenation (the `||`/`CONCAT`/`+` pattern from the DBMS cheat-sheet). No skill gap.
