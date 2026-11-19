---
title: "Exploiting an SQL injection with WAF bypass"
author: Benoit Philippe (Vaadata)
url: https://www.vaadata.com/en/blog/exploiting-an-sql-injection-with-waf-bypass/
date_published: 2022-12-13
date_scraped: 2026-11-19
platform: other
disclosed_report: none
bounty: not stated (pentest)
vuln_class: sqli
---

<!-- Raw scraped text below. Public writeup. Condensed. MySQL target behind a WAF. -->

# WAF bypass by moving the injection to a less-inspected input (Vaadata)

Pentest. Burp Scanner flagged a possible SQLi on the `idEntity` GET parameter, but sqlmap found nothing. The lesson: when a WAF blocks one input, move the injection to an input the WAF does not inspect.

## Confirm it is not a false positive (manual boolean-blind)
Legit: `?idEntity=442` -> response contains `NameEntity="Rubeus Agrid"`; `?idEntity=0` (invalid) -> `NameEntity=""`.
Burp's probe requests showed boolean-blind working:
```
?idEntity=44252262402 or 2346=2346   -> NameEntity="John Smith"  (=> or 1 => true, returns first entity id 1)
?idEntity=44291517249 or 6362=6367   -> NameEntity=""            (=> or 0 => false, invalid id, empty)
```
So the operators `or`/`=` are interpreted: the boolean-blind SQLi is real.

## Why sqlmap failed: a WAF
Through a Burp proxy, every sqlmap payload returned an empty `HTTP 200, Content-Length: 0`. A manual `?idEntity=442 and SLEEP(5)--` also returned an empty 200 (no delay): the WAF blocks attack payloads on the GET parameter.

## Partial bypasses tried
- `-A "NONE"` (change the User-Agent off `sqlmap/...`): sqlmap now DETECTS the boolean-blind injection (`idEntity=442 AND 4465=4465`), but data extraction still fails because the WAF blocks `SELECT`, `SLEEP`, `FROM`, `WHERE` keywords.
- `--tamper` scripts: none of the built-in tamper scripts worked here.

## The actual bypass: a different, less-inspected input
Burp's **ParamMiner** fuzzed for hidden params and found an `idEntitySelected` **cookie** that drives the same query (used when the `idEntity` GET param is absent). The WAF is strict on GET params but lax on cookie values:
```
Cookie: idEntitySelected=1+or+SLEEP(5)--   -> response delayed 5s (WAF did NOT block it)
```
Point sqlmap at the cookie:
```
sqlmap -u https://example.com/endpoint --cookie='idEntitySelected=1' --level=2 \
  --param-filter=cookie --not-string='nameEntity = "";' -A "NONE"
```
Flags: `--level=2` (needed to test cookies), `--param-filter=cookie` (only cookies), `--not-string` (marks the failed-query signature). Result: boolean-blind, time-based (`AND (SELECT 6499 FROM (SELECT(SLEEP(5)))x)`), AND a 1-column UNION all work via the cookie, enabling full DB enumeration.

## Lessons
- Burp flags it, sqlmap says no -> do not give up; confirm manually, then find out why (often a WAF returning empty/blocked responses).
- A WAF usually inspects some inputs more than others. If the GET param is guarded, test the same value delivered via a **cookie or header** (ParamMiner finds hidden ones).
- sqlmap aids: `--technique`, `--not-string`/`--string`, `--proxy` (to watch traffic), `-A` (User-Agent), `--tamper`, `--cookie`/`--level`/`--param-filter`.
