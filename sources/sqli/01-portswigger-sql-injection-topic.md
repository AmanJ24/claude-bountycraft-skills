---
title: "SQL injection (topic)"
author: PortSwigger Web Security Academy
url: https://portswigger.net/web-security/sql-injection
date_scraped: 2026-11-19
platform: portswigger
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public reference. The spine reference for the skill. -->

# SQL injection

SQL injection (SQLi) lets an attacker interfere with the queries an application makes to its database: view data belonging to other users or that the app can access, modify or delete it (persistent changes), and in some cases escalate to compromise the underlying server or run denial-of-service. High-profile breaches, reputational damage, regulatory fines, and persistent backdoors have resulted.

## How to detect SQLi (manual, per entry point)
- Submit the single quote `'` and look for errors or anomalies.
- Submit SQL syntax that evaluates to the base value vs a different value; look for systematic response differences.
- Boolean conditions `OR 1=1` and `OR 1=2`; look for response differences.
- Payloads that trigger time delays; look for timing differences.
- OAST payloads that trigger out-of-band network interactions; monitor for interactions.
Burp Scanner finds most SQLi quickly and reliably.

## SQLi in different parts of the query
Most SQLi occurs in the `WHERE` clause of a `SELECT`, but it can occur anywhere:
- `UPDATE` statements: in the updated values or the `WHERE` clause.
- `INSERT` statements: in the inserted values.
- `SELECT`: within the table or column name.
- `SELECT`: within the `ORDER BY` clause.

## Examples

### Retrieving hidden data
`https://insecure-website.com/products?category=Gifts` runs `SELECT * FROM products WHERE category = 'Gifts' AND released = 1`.
- `category=Gifts'--` comments out the rest: `... WHERE category = 'Gifts'--' AND released = 1` (shows unreleased products; `--` is a SQL comment).
- `category=Gifts'+OR+1=1--` returns all items because `1=1` is always true.
- Warning: `OR 1=1` can reach an UPDATE/DELETE if the input is reused in multiple queries, causing data loss. Take care.

### Subverting application logic (login bypass)
`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`. Submitting username `administrator'--` and blank password yields `... WHERE username = 'administrator'--' AND password = ''`, logging in as administrator with no password.

### Retrieving data from other tables (UNION)
If the app runs `SELECT name, description FROM products WHERE category = 'Gifts'`, the input `' UNION SELECT username, password FROM users--` appends all usernames and passwords.

## Blind SQLi
When the app returns neither query results nor DB errors. Exploited by: (a) conditional responses (inject a condition that changes a detectable response, e.g. a boolean or a conditional error like divide-by-zero), (b) conditional time delays (infer truth from response time), (c) out-of-band (OAST) network interactions, often exfiltrating data directly, e.g. in a DNS lookup to a domain you control. OAST is powerful and works where the others do not.

## Second-order (stored) SQLi
First-order: input from a request is unsafely placed into a query immediately. Second-order: input is safely stored (no vuln at storage), then later retrieved and incorporated into a query unsafely, because the developer wrongly deems stored data trusted. Also known as stored SQLi.

## Examining the database
Core SQL features work the same across platforms, but many differ (string concatenation syntax, comments, batched/stacked queries, platform APIs, error messages). After finding SQLi, obtain DB info:
- Version: method differs by DB type, so a working method reveals the DB type. Oracle: `SELECT * FROM v$version`.
- List tables: `SELECT * FROM information_schema.tables` (most DBs).
See the SQL injection cheat sheet and "Examining the database" page.

## SQLi in different contexts
Any controllable input processed as SQL can be an injection point, including JSON or XML bodies. Different formats can obfuscate attacks blocked by WAFs. Weak filters look for SQL keywords in the request; encoding/escaping characters in the keyword can bypass them. Example XML-based SQLi encoding the `S` in `SELECT`:
```
<stockCheck><productId>123</productId>
<storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId></stockCheck>
```
Decoded server-side before passing to the SQL interpreter. (Lab: SQL injection with filter bypass via XML encoding.)

## Prevention
Use parameterized queries (prepared statements), never string concatenation. Vulnerable:
```
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
```
Safe:
```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
```
Parameterized queries work for data in WHERE/INSERT/UPDATE values, but NOT for table/column names or `ORDER BY`; for those, whitelist permitted values or use different logic. The query string must be a hard-coded constant, never variable data; do not decide case-by-case whether data is "trusted".
