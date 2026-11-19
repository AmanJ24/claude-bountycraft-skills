# DoD: boolean-blind SQLi in the User-Agent header (MySQL 8 / SharePoint)

- **Vuln type / sub-variant:** boolean-based blind SQLi where the injection point is the HTTP `User-Agent` **header**, not a parameter
- **Target type:** a .mil app on Windows / Microsoft SharePoint 16.0.0.5452 with a MySQL 8 (MariaDB fork) backend that logs/queries the UA
- **Recon signal:** the app stores the User-Agent in the DB; a boolean appended to the UA changes the response
- **Root cause:** the User-Agent header value is concatenated into a query (common when UA is logged)
- **Exploitation steps:**
  1. Append a boolean to a normal UA: `...Safari/523.10' AND 8074=8074-- KwOG` and compare to a false variant; the response differs.
  2. Automate: `sqlmap --url https://[REDACTED].mil/ --random-agent --risk 3 --level 5 --batch` (level 5 is required for sqlmap to test the UA header; `--random-agent` rotates UAs).
- **Bypasses used:** none; the point is the header context; sqlmap needs high `--level` to reach headers
- **Impact:** information disclosure, inferring DB schema and data one boolean at a time; rated Medium; resolved
- **Bounty:** VDP (likely none)
- **Lesson / pattern:** headers (User-Agent, Referer, X-Forwarded-For, Cookie) are injection points when the app stores/queries them. sqlmap only tests headers at higher `--level` (3 for Cookie, 5 for UA/Referer), so low-level scans miss them. Confirm boolean-blind by diffing `' AND <true>-- -` vs `' AND <false>-- -` in the header.
- **Source:**
  - Title: Boolean Based Blind SQL Injection Via User-Agent in a .mil site
  - Author: iamunixtz (reporter), U.S. Dept Of Defense (program)
  - URL: https://hackerone.com/reports/2599826
  - Date: 2025-01-24 (scraped 2026-11-19)
  - Raw file: `sources/sqli/18-h1-2599826-dod-useragent-blind-sqli.md`

Tags: boolean-blind, header-context, user-agent, mysql, sqlmap, sqlmap-level, information-disclosure, medium, source-real
