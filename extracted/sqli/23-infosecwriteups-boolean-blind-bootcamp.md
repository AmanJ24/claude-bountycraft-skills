# Boolean-blind as a "20 questions" oracle: username-checker example, information_schema + binary search

- **Vuln type / sub-variant:** boolean-based blind SQLi, end-to-end data extraction from a binary (true/false) oracle
- **Target type:** any endpoint returning a binary signal tied to a DB condition: a signup "username taken/available" checker, login success/failure, CAPTCHA verification, or a differing HTTP status
- **Recon signal:** a feature that answers only yes/no based on the database (e.g. `{"taken": true}` for `admin`, `{"taken": false}` for a random name)
- **Root cause:** the username (or similar) is concatenated into `SELECT * FROM users WHERE username = '<input>'`, and the app reflects whether a row returned
- **Exploitation steps:**
  1. Confirm: `admin' AND 1=1-- -` -> taken:true; `admin' AND 1=2-- -` -> taken:false. The flip proves the oracle evaluates your logic.
  2. Ask structured yes/no questions with subqueries + `LIKE`/`SUBSTRING`/`MID`.
  3. Enumerate DB names: count with `(SELECT COUNT(*) FROM information_schema.schemata)=N`, then each char via ASCII binary search `(SELECT ASCII(SUBSTRING(schema_name,1,1)) FROM information_schema.schemata LIMIT 0,1) > 97`.
  4. Tables/columns via `information_schema.tables/columns ... table_name LIKE 'u%'` drilled to `users`, columns `LIKE 'p%'` -> `password`.
  5. Extract data: find `LENGTH(password)=N` first, then each char by ASCII binary search.
- **Bypasses used:** none; pure inference. Optimization: try predictable names (`username`,`password`,`email`) before brute forcing to save requests
- **Impact:** reconstruct entire databases/tables/records one char at a time from a single boolean bit; exploits apps that look "secure" because they do not leak data directly
- **Bounty:** not stated (bootcamp writeup)
- **Lesson / pattern:** any binary response tied to DB content is an oracle. Confirm with `AND 1=1` vs `AND 1=2`, map structure with `information_schema` + `LIKE`, then length-then-char binary search. Automate with Burp Intruder / sqlmap but know the mechanics for when sqlmap fails.
- **Source:**
  - Title: Bug Bounty Bootcamp #28: Boolean-Based Blind SQL Injection
  - Author: Aman Sharma (InfoSec Write-ups)
  - URL: https://infosecwriteups.com/bug-bounty-bootcamp-28-boolean-based-blind-sql-injection-extracting-data-one-true-false-at-a-abe69e3de702
  - Date: 2026-03-10 (member-only; user-supplied full article)
  - Raw file: `sources/sqli/23-infosecwriteups-boolean-blind-bootcamp.md`

Tags: boolean-blind, blind, oracle, information-schema, substring-extraction, binary-search, like-operator, mysql, sqlmap, methodology, source-real
