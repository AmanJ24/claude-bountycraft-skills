# XSS filter evasion cheat sheet (OWASP)

Reference payload-bank cheat sheet. Catalogs the categories of filter evasion that defeat naive input filters and blacklists. Captured here as technique categories with a single minimal representative token each, not the full payload bank.

- **Vuln type / sub-variant:** Reflected/stored XSS where a filter or blacklist tries to strip or detect script markup. The cheat sheet enumerates evasion categories that slip past such filters.
- **Target type:** Any app relying on string-level input filtering (blacklists, naive tag/keyword removal, SRC-domain checks) rather than context-aware output encoding.
- **Recon signal:** A filter that blocks some obvious vector (`<script>`, `javascript:`, `onerror`) but is implemented as pattern matching, suggesting a parser/filter differential the browser will still execute.
- **Root cause:** Browsers are far more permissive than filters. HTML parsing tolerates malformed markup, many encodings, whitespace, and alternate event-handler/tag entry points, so a filter that normalizes or matches differently than the browser can be evaded.
- **Evasion categories (minimal representative token each):**
  1. Case variation: mixed-case tags/handlers defeat case-sensitive matching. Token: `<ScRiPt>`.
  2. Encoding: decimal/hex HTML entities (with or without trailing semicolon), URL/UTF/US-ASCII, and `String.fromCharCode` to hide keywords. Token: `&#x6A;` for `j`.
  3. Quote-less / attribute tricks: attributes and `src` values without quotes, empty/absent `src`, or default-src tags to dodge domain checks. Token: `<img src=x onerror=...>`.
  4. Event handlers: non-script execution via a large set of inline handlers (`onerror`, `onload`, `onfocus`, etc.) on many tags. Token: `onerror=`.
  5. Embedded whitespace / control chars: tabs, newlines, carriage returns, and null bytes inserted inside `javascript:` or between tokens to break signatures. Token: `java\tscript:`.
  6. Malformed / half-open tags: extraneous open brackets, missing closing `</script>`, half-open HTML, protocol-relative script sources. Token: `<img """><svg onload=...>`.
  7. Alternate sinks/tags: `svg`, `object`, `embed`, `iframe`, `meta` refresh, `base`, `style`/CSS expressions, `body` background, and framework/VBScript legacy vectors. Token: `<svg onload=...>`.
  8. URL-string evasion: IP-as-DWORD/hex/octal, mixed encoding, protocol-resolution bypass, and HTTP parameter pollution to smuggle vectors. Token: decimal/hex IP forms.
- **Bypasses used:** the above are the bypasses, plus a dedicated character-escape-sequence table and a WAF-bypass section (alert obfuscation, request redirection, DOM-based variants).
- **Impact:** Script execution despite input filtering; the polyglot locator fires across multiple injection contexts.
- **Bounty:** n/a (reference).
- **Lesson / pattern:** Blacklisting and tag stripping cannot keep up with browser parsing. Defend with context-aware output encoding and a vetted sanitizer, not keyword filters; every category here exploits a filter/parser differential.
- **Source:**
  - Title: XSS Filter Evasion Cheat Sheet
  - Author: OWASP (Cheat Sheet Series)
  - URL: https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html
  - Date: not stated
  - Disclosed report: none
  - Raw file: `sources/xss/29-owasp-xss-filter-evasion.md`

Tags: filter-bypass, encoding, polyglot, waf-bypass, methodology
