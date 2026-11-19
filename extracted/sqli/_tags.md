# Canonical tags (SQL Injection)

Normalized 2026-11-19. Use these spellings in `Tags:` lines.

- Channel (how results come back): in-band, union-based, error-based, boolean-blind, time-based, oob-oast, blind
- Sub-variant / class: second-order, stacked-queries, auth-bypass-tautology, logic-subversion
- Injection context (where the input sits): where-clause, order-by, insert, update, url-path, query-param, post-param, header-context, user-agent, cookie-context, json-body, graphql, set-session
- Technique: union-column-count, union-concat, string-column, substring-extraction, binary-search, conditional-error, cast-error, extractvalue, information-schema, dbms-fingerprint, xor-payload, pg-sleep, dns-exfiltration
- DBMS: mysql, mariadb, postgresql, mssql, oracle, sqlite
- WAF / evasion: waf-bypass, json-operator, dialect-syntax, comment-whitespace, newline, parentheses, case-variation, versioned-comment, less-inspected-input, sqlmap-level, false-positive-triage
- Escalation / impact: xp-cmdshell, udf, plpythonu, java-stored-proc, copy-to-program, into-outfile, load-file, xp-dirtree, file-read, file-privilege, rce, privilege-escalation, auth-bypass, full-db-dump, data-exfiltration, credential-disclosure, pii, information-disclosure, cvss-10, critical, medium
- Tooling: sqlmap, ghauri, burp-collaborator, interactsh, paramminer, curl, os-shell, responsible-poc
- Meta: reference, methodology, taxonomy, cheat-sheet, defense, source-real, source-partial, scanner-blind-spot, research

Notes:
- The spine question: "where does my input land in the query, and how does the app reveal the result?" Context (where-clause / order-by / insert / header / json / graphql) decides the breakout; the revealed signal (reflected output / error / timing / nothing) decides the channel (union / error / boolean / time / oob).
- `parameterized-queries` / `defense` mark the universal fix; `dbms-fingerprint` marks the pivot from detection to DBMS-specific exploitation.
- References / methodology (no single finding): 01 (PS spine), 02 (UNION), 03 (blind), 04 (DBMS cheat-sheet), 06 (YesWeHack bug-bounty methodology), 07 (Claroty JSON-WAF), 20 (0x3adly SQLi->RCE DBMS reference).
- Real disclosed/pentest findings: 12 (H1 GraphQL, Critical), 13 (H1 Automattic UNION-in-path), 15 (H1 Acronis POST param), 18 (H1 DoD User-Agent header), 19 (Velstadt MSSQL->RCE), 21 (NetSPI second-order OOB), 22 (Vaadata WAF-via-cookie), 24 (boumriga time-based + ghauri).
- Scope: classic SQLi only. NoSQL injection (`{"$ne":...}`, `$where`, `$regex`) is deferred to a future `nosqli` skill and only cross-referenced for the login-bypass class. ORM-layer injection noted as a variant.
- Pending backfill: #23 (InfoSec boolean-blint bootcamp, member-only) once the user downloads it to `Manual_downloads/`; it adds a boolean "username taken/available" oracle case (technique already covered by 03/18/22/24).
