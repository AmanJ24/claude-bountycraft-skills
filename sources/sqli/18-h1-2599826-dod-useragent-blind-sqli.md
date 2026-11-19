---
title: "Boolean Based Blind SQL Injection Via User-Agent in a .mil site"
author: iamunixtz (reporter), U.S. Dept Of Defense (program)
url: https://hackerone.com/reports/2599826
date_published: 2025-01-24
date_scraped: 2026-11-19
platform: hackerone
disclosed_report: https://hackerone.com/reports/2599826
bounty: Hidden (VDP, likely none)
vuln_class: sqli
---

<!-- Raw scraped text below. Public, disclosed report. Severity Medium. Target redacted (.mil). -->

# Boolean-blind SQLi in the User-Agent header (DoD, MySQL/SharePoint)

A boolean-based blind SQL injection in the `User-Agent` HTTP **header** of a .mil application. The injection point is not a URL/body parameter but a header value that the app logs/queries. Severity Medium; resolved.

## Environment (fingerprinted)
- DBMS: MySQL 8 (MariaDB fork)
- Web server OS: Windows
- App tech: Microsoft SharePoint 16.0.0.5452

## PoC
The `User-Agent` header carries the payload. Boolean test appended to a normal UA string:
```
User-Agent: Mozilla/5.0 (Macintosh; U; PPC Mac OS X; en-us) AppleWebKit/523.10.3 (KHTML, like Gecko) Version/3.0.4 Safari/523.10' AND 8074=8074-- KwOG
```
The app responds differently depending on the boolean (`8074=8074` true vs a false variant), confirming boolean-blind SQLi. Automated with sqlmap:
```
sqlmap --url https://[REDACTED].mil/ --random-agent --risk 3 --level 5 --batch
```
(`--level 5` is required for sqlmap to test the User-Agent header; `--random-agent` rotates UAs. Burp can also be used to craft boolean payloads manually in the header.)

## Impact
Information disclosure: infer DB schema and sensitive data one boolean at a time.

## Lesson
- HTTP headers (User-Agent, Referer, X-Forwarded-For, Cookie) are injection points when the app stores/queries them. Apps often log the UA into a DB.
- sqlmap tests headers only at higher `--level` (3 for Cookie, 5 for User-Agent/Referer); a low-level scan misses header-context SQLi.
- Boolean-blind confirmation: append `' AND <true>-- -` vs `' AND <false>-- -` to the header and diff the responses.
