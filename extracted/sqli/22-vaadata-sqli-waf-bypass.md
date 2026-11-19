# WAF bypass by moving the injection to a less-inspected input (a cookie the WAF ignores)

- **Vuln type / sub-variant:** boolean/time/UNION SQLi behind a WAF, bypassed by switching the injection from a guarded GET parameter to a lax cookie; MySQL
- **Target type:** an endpoint whose `idEntity` GET param drives a query; the same query is also reachable via an `idEntitySelected` cookie
- **Recon signal:** Burp Scanner flags SQLi on `idEntity` but sqlmap finds nothing; attack payloads on the GET param return an empty `200, Content-Length: 0` (a WAF)
- **Root cause:** the WAF strictly inspects GET params but is lax on cookie values; the backend query is the same
- **Exploitation steps:**
  1. Confirm manually it is not a false positive: `?idEntity=...+or+2346=2346` returns the first entity (true), `...+or+6362=6367` returns empty (false).
  2. Diagnose the WAF: proxy sqlmap through Burp; every attack payload returns empty 200; a manual `and SLEEP(5)--` also empty -> WAF blocking.
  3. Partial: `-A "NONE"` (change the sqlmap User-Agent) gets detection but extraction still blocked (WAF drops SELECT/SLEEP/FROM); tamper scripts failed.
  4. Pivot input: Burp ParamMiner finds an `idEntitySelected` cookie driving the same query; `Cookie: idEntitySelected=1+or+SLEEP(5)--` delays 5s (not blocked).
  5. sqlmap via the cookie: `--cookie='idEntitySelected=1' --level=2 --param-filter=cookie --not-string='nameEntity=""' -A "NONE"` yields boolean, time, and UNION; full enumeration.
- **Bypasses used:** switching to a less-inspected input (cookie); sqlmap `-A`, `--not-string`, `--proxy`, `--param-filter=cookie`, `--level=2`
- **Impact:** full DB enumeration once the WAF is bypassed
- **Bounty:** not stated (pentest)
- **Lesson / pattern:** Burp says yes, sqlmap says no -> confirm manually and look for a WAF (empty/blocked responses). WAFs inspect some inputs more than others: if the GET param is guarded, deliver the same value via a cookie or header (ParamMiner finds hidden ones). Know the sqlmap WAF flags (`-A`, `--tamper`, `--not-string`, `--param-filter`).
- **Source:**
  - Title: Exploiting an SQL injection with WAF bypass
  - Author: Benoit Philippe (Vaadata)
  - URL: https://www.vaadata.com/en/blog/exploiting-an-sql-injection-with-waf-bypass/
  - Date: 2022-12-13 (scraped 2026-11-19)
  - Raw file: `sources/sqli/22-vaadata-sqli-waf-bypass.md`

Tags: waf-bypass, less-inspected-input, cookie-context, boolean-blind, time-based, union-based, mysql, sqlmap, paramminer, false-positive-triage, source-real
