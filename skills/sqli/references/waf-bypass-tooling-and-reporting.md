# WAF/filter bypass, tooling, and reporting

Full detail behind SKILL.md sections 4 and 6. From notes 06, 07, 15, 18, 19, 22, 24.

## WAF / filter bypass
When a payload is blocked (often an empty `200, Content-Length: 0`, a 403, or a dropped keyword):

1. **Keyword/whitespace obfuscation (note 06):**
   - Case variation: `SeLeCT` (SQL keywords are case-insensitive).
   - Inline comments for spaces: `SELECT/**/username/**/FROM/**/users`.
   - Newlines for spaces: `'OR%0A1=1-- -`.
   - Parentheses: `'OR(1=1)-- -` (evades a filter matching `OR 1=1`).
   - MySQL versioned comments execute: `/*!SELECT*/`, `/*!50000SELECT*/`.
   - Encoding: URL-encode keywords, build strings with `CHAR()`/`0x..` hex, or XML entities (`&#x53;ELECT`) in XML/JSON bodies.
2. **JSON-operator bypass (note 07):** append a valid-but-unmodelled JSON operator so the WAF's SQL parser chokes while the DB executes. PostgreSQL `@>`, `<@`, `->`, `->>`, `?`, `?|`, `?&`. Generalizes to any dialect syntax the WAF under-models (the same idea works for other uncommon-but-valid syntax).
3. **Move to a less-inspected input (note 22):** WAFs inspect inputs unevenly. If the GET param is guarded, deliver the same value via a **cookie** or **header** (Burp ParamMiner finds hidden params/cookies). In the Vaadata case the WAF blocked the GET param but ignored an `idEntitySelected` cookie that drove the same query.
4. **sqlmap tamper scripts:** `--tamper=space2comment,between,charencode,...` (`sqlmap --list-tamper`). Not a silver bullet (note 22: none of the built-ins worked, the cookie pivot did).

## Tooling
- **Confirm manually first** (curl/Repeater) so the finding and report are credible (notes 22, 24).
- **sqlmap:**
  - `-r <request.txt>` to inject from a full raw HTTP request; `-p <param>` to target one (note 15).
  - `--level` reaches more inputs: 2 cookies, 3 cookies+UA, 5 UA/Referer/host headers (note 18); `--risk` adds heavier payloads.
  - `--technique=B/E/U/T/S`, `--dbms=<engine>`, `--not-string`/`--string` to mark the true/false signature (note 22), `--proxy=http://127.0.0.1:8080` to watch traffic in Burp, `-A` to set the User-Agent, `--os-shell` for RCE.
  - JSON bodies: `--data='{"x":"*"}'` with `*` marking the injection point (note 19).
- **ghauri:** better for time-based blind on unstable/WAF-ish targets (`--technique T --time-sec 20 --delay 20 --retries 10`), note 24.
- **OAST:** Burp Collaborator or Interactsh to capture DNS/HTTP callbacks for blind/OOB.
- **Burp ParamMiner:** discover hidden params/headers/cookies to pivot the injection.

## Reporting
- **Name the input and context:** "the `s` GET param is injected into a MySQL WHERE clause", "the User-Agent header is logged into a query", "the GraphQL `embedded_submission_form_uuid` parameter reaches a SET SESSION statement". Give the exact request.
- **Prove minimally and non-destructively:** a scaling time delay (5/10/15s), a boolean true/false pair, an OAST DNS hit, or `current_user()`/`@@version`. Responsible PoC: `sqlmap -r <request> -p <param> --current-user` and stop (notes 15, 24). Do not dump real users' data or run OS commands outside an authorized engagement.
- **State impact and privilege:** what data is readable (and whose), auth bypass, data modification; the DBMS and the account's privilege (a `sa`/`root`/`FILE`/DBA account means file read/write or RCE is reachable, note 19). Severity: Medium (info disclosure) to Critical (full dump / auth bypass / OS RCE).
- **Be honest about the channel and preconditions:** blind extraction is slow but real; OOB needs egress; RCE needs a privileged DB account. Note whether the finding is unauthenticated.
- **Remediation to recommend:** parameterized queries / prepared statements (primary); for identifiers/ORDER BY use allowlists; least-privilege DB account; suppress verbose errors; disable and lock down `xp_cmdshell`; WAF as a secondary layer only (it can be bypassed, as above).
