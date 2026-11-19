# case-05: SQLi with filter bypass via XML encoding (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding
- **Skill path:** SKILL.md section 4 (filter/WAF bypass via encoding); references/waf-bypass-tooling-and-reporting.md; notes 06, 07.
- **Date:** 2026-11-19, agent-run live.

## What the skill told us to do
When a keyword filter/WAF blocks SQL, encode the keyword characters so the filter cannot match, while the parser decodes them before SQL. XML/JSON bodies allow entity encoding (note 01/06 the `&#x53;ELECT` idea).

## Steps
1. Stock-check endpoint `POST /product/stock` with an XML body; `storeId` is injectable.
2. Plain `<storeId>1 UNION SELECT username || '~' || password FROM users</storeId>` -> **403 "Attack detected"** (keyword filter).
3. Hex-entity-encoded every character of the payload (`&#x55;&#x4e;&#x49;&#x4f;&#x4e;...` for `UNION...`) so the filter saw only entities; the XML parser decoded them before the SQL layer. Request passed and returned `administrator~<password>`.
4. Logged in as administrator.

## Result
**Green banner: "LAB Solved"**. Directly validated the encoding-based WAF/filter bypass (note 06/07). Confirmed the diagnostic that a blocked plain payload + a working encoded one proves a keyword filter, not a non-injectable endpoint. No skill gap.
