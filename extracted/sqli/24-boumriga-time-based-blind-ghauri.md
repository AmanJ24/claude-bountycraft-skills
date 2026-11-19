# Time-based blind SQLi: curl baseline-and-scale confirmation, then ghauri extraction (MySQL)

- **Vuln type / sub-variant:** time-based blind SQLi; MySQL; unauthenticated search parameter
- **Target type:** a search `?s=` GET parameter passed unsanitized into a MySQL query
- **Recon signal:** no visible output or errors, but a sleep payload delays the response proportionally
- **Root cause:** the `s` value concatenated into a MySQL query; synchronous execution means a DB sleep delays the HTTP response
- **Exploitation steps:**
  1. Payload: `'XOR(if(now()=sysdate(),SLEEP(N),0))XOR'Z` (`now()=sysdate()` is ~always true, so `SLEEP(N)` runs; `XOR` wrappers fix the string context; trailing `Z` closes it).
  2. Confirm with curl by scaling N: baseline ~0.5s; `SLEEP(5)` -> ~5.3s; `SLEEP(10)` -> ~10.3s; `SLEEP(15)` -> ~15.3s. Linear, consistent scaling is proof (a single delay could be noise).
  3. Extract with ghauri: `ghauri -u "https://[REDACTED]/?s=test" --dbms mysql --technique T --banner --batch --level 3 --confirm --retries 10 --flush-session --time-sec 20 --delay 20` -> banner, hostname, current DB, current user.
- **Bypasses used:** `XOR`-wrapped payload for the string context; ghauri's fine timing control (`--time-sec`, `--delay`, `--retries`) for unstable/WAF-ish targets where sqlmap struggles
- **Impact:** extract schema/tables/columns/rows by inference, enumerate users/privileges, map infra; dump all data given enough requests; file read/write if the DB user has `FILE`
- **Bounty:** not stated (active HackerOne program, redacted)
- **Lesson / pattern:** always confirm manually with curl before tooling, and insist on linearly scaling delays (not a single slow response) as proof. ghauri is strong for time-based blind on unstable targets; raise `--time-sec`/`--delay` on slow or WAF-protected targets to avoid false negatives.
- **Source:**
  - Title: Time-Based Blind SQL Injection: Confirming and Exploiting with curl and Ghauri
  - Author: Bahmed Boumriga
  - URL: https://medium.com/@boumriga.bahmed/time-based-blind-sql-injection-confirming-and-exploiting-with-curl-and-ghauri-3c7bf94aaffd
  - Date: 2026-06-20 (scraped 2026-11-19)
  - Raw file: `sources/sqli/24-boumriga-time-based-blind-ghauri.md`

Tags: time-based, blind, mysql, xor-payload, curl, ghauri, dbms-fingerprint, file-privilege, source-real
