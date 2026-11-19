# Detecting SQLi and working the injection context

Full detail behind SKILL.md sections 1-3. The two questions: where does my input land in the query (context -> breakout), and how does the app reveal the result (channel -> technique).

## Detection probes (run on every entry point)
From notes 01, 06:
- `'` (single quote): DB error, 500, or a changed/empty response. The classic first probe. Try `"` and backtick too.
- Boolean: `' OR 1=1--` vs `' OR 1=2--`, and the safer `' AND 1=1--` vs `' AND 1=2--` (prefer AND; a blanket OR can hit UPDATE/DELETE).
- Arithmetic/string that evaluates to the same vs a different value (e.g. `?id=2` vs `?id=1+1` vs `?id=3-1`) and compare.
- Time delay (DBMS-specific): MySQL `SLEEP(5)`, PG `pg_sleep(5)`, MSSQL `WAITFOR DELAY '0:0:5'`, Oracle `dbms_pipe.receive_message(('a'),5)`.
- OAST: a payload that triggers a DNS/HTTP callback you monitor (Burp Collaborator / Interactsh).
- Read the error: a verbose message that quotes the query ("Unclosed quotation mark...", "PG::SyntaxError", "You have an error in your SQL syntax") both confirms and maps the context.

## Determine the context (what to break out of)
- **Quoted string in WHERE** (most common): close with `'`, then comment the tail (`-- -`, `#`, `/* */`). A verbose error like `... WHERE id = '''` confirms a single-quoted string.
- **Numeric in WHERE**: no quote needed; `?id=2 OR 1=1`, `?id=2-0`, `?id=2 AND SLEEP(5)`.
- **ORDER BY**: cannot use a parameter; you can inject `ORDER BY (CASE WHEN (cond) THEN 1 ELSE 2 END)` or a subquery. Parameterization does not protect this clause (note 01).
- **INSERT/UPDATE values**: a `'` breaks the tuple; a classic `x' OR 1=1 -- -` fails (wrong column count). Use `x'||'y`, or close the tuple: `x', (SELECT ...), 'y')-- -` (note 06). Be careful: UPDATE/DELETE injections can change data.
- **SET SESSION / non-WHERE contexts** (note 12): e.g. `SET SESSION <key> TO '<injection>'`; scanners expecting WHERE miss these, so test manually.
- **LIMIT / GROUP BY / HAVING**: DBMS-specific breakout; often needs subqueries or stacked queries.

## Injection points to fuzz (not just ?params)
- Query parameters, search fields (`?id=`, `?category=`, `?s=`).
- URL path segments (`/resource/<id>`), note 13.
- POST body fields, including obscure ones (`log`), note 15.
- JSON body fields and GraphQL variables/parameters (notes 12, 19); JSON also enables obfuscation (encode a keyword char).
- HTTP headers the app logs/queries: `User-Agent`, `Referer`, `X-Forwarded-For`, `Cookie` (note 18). sqlmap tests Cookie at `--level 3`, UA/Referer at `--level 5`.
- Stored fields reused in a later request (second-order): username, display name, preferences, report params.

## Pick the channel by the revealed signal
| App reveals | Channel | Reference |
|---|---|---|
| Query output in the response | UNION (or error-based) | `in-band-union-and-error.md` |
| A catchable DB error | error-based (CAST/EXTRACTVALUE/conditional) | `in-band-union-and-error.md` |
| A true/false behavioural difference | boolean-blind | `blind-oob-and-second-order.md` |
| Only timing changes | time-based | `blind-oob-and-second-order.md` |
| Nothing / async | out-of-band (OAST/DNS) | `blind-oob-and-second-order.md` |
| Stored then used later | second-order (any channel, usually OOB) | `blind-oob-and-second-order.md` |

## Fingerprint the DBMS early
A method that works on only one engine identifies it: string concat (`||` vs `+` vs `CONCAT`), version (`@@version` vs `version()` vs `v$version`), time function, comment style. Once known, pull exact syntax from `dbms-specific-and-escalation.md`.

## Confirm, do not assume
- Require a provable signal: a scaling time delay, a boolean flip, an OAST hit, or reflected injected data. A lone 500 or reflected value is not proof (SKILL.md section 5).
- Keep it non-destructive: prefer `AND`-based booleans over `OR 1=1`; extract `current_user`/version as proof, not real data.
