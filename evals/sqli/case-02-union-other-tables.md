# case-02: UNION attack, retrieving data from other tables (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables
- **Skill path:** SKILL.md section 3 (UNION); references/in-band-union-and-error.md; note 02.
- **Date:** 2026-11-19, agent-run live.

## Steps
1. Injection point `/filter?category=`; original query returns 2 string columns.
2. `' UNION SELECT username, password FROM users--` returned the `users` table in the product listing; parsed `administrator` and its password.
3. Logged in as administrator with the extracted password.

## Result
**Green banner: "LAB Solved"**. Textbook UNION: matched column count, both columns string, read another table. No skill gap.
