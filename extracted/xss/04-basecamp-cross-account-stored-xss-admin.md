# Basecamp: cross-account stored XSS via attachment content, sanitizer and nonce-propagation chain, persistent admin takeover

- **Vuln type / sub-variant:** Cross-account stored XSS (mutation-XSS flavored). Attacker HTML smuggled in an attachment `content` attribute, reparsed by the editor (parser normalization), survives DOMPurify via allowed data attributes, reaches `innerHTML`, then escalates to script execution by abusing CSP-nonce propagation to a cross-origin Turbo frame.
- **Target type:** Rich-text message editor (Lexxy) in a SaaS collaboration app (app.basecamp.com), stored message with a `bc-attachment` element.
- **Recon signal:** A `bc-attachment` with a valid server-generated `sgid` has its client-supplied `content` attribute trusted and reparsed when the message is opened in the Edit form. The hunter noticed that invalid nested HTML inside that attribute gets normalized by the parser, and that `data-behavior`/`data-url` attributes were not stripped.
- **Root cause:** Four compounding failures. (1) Client-supplied `bc-attachment.content` is trusted because the attachment `sgid` resolves. (2) HTML parsing is not a sanitization boundary: an invalid `select > option > div` sequence is normalized by the parser into a live `div`, exposing attributes that become active (parser reparse / mutation). (3) DOMPurify permits `data-behavior="load_drafts_link"` and attacker-controlled `data-url` (ALLOW_DATA_ATTR left on). (4) A legacy behavior sinks the fetched response to `innerHTML`. Unescaped sink: attacker-fetched HTML assigned via `element.innerHTML`. The CSP that should have blocked inline script is defeated because Turbo copies the page's CSP nonce onto the script inside an attacker-controlled cross-origin Turbo frame. CSRF protection fails because the injected script reads the victim's own valid token from the same origin.
- **Exploitation steps:**
  1. As an outside collaborator (`O`), create a normal YouTube embed to obtain a valid `sgid`.
  2. Post a message in the victim project with a `bc-attachment` carrying that `sgid` and an encoded nested `select>option>div` with `data-behavior="load_drafts_link"` and `data-url` pointing at the attacker host. The reader view renders as a normal embed.
  3. A victim account administrator (`A`) opens the message's Edit form once. Lexxy normalizes the markup, the surviving data attributes activate the legacy behavior.
  4. The behavior fetches the attacker `/drafts` response and assigns it via `innerHTML`; that response injects a cross-origin `<turbo-frame>`.
  5. Turbo loads the attacker `/frame` response and applies A's CSP nonce to the inline script, so it executes on app.basecamp.com.
  6. The script reads A's CSRF token from page metadata and POSTs to the administrator-grant endpoint as A, promoting O to administrator.
- **Bypasses used:** Parser normalization of invalid nesting to expose live attributes; DOMPurify data-attribute allowlist gap; CSP-nonce propagation to a cross-origin Turbo frame (CSP bypass); CSRF defeated by same-origin token reuse; attachment-content trust keyed only on a resolvable `sgid`.
- **Impact:** Arbitrary JS in an account owner/admin session on one Edit-form open, escalated to persistent administrator control of the victim account (add/remove admins, manage people and project access, read account-management records, retention settings). The grant survives after the victim browser closes. Demonstrated by promoting a second member.
- **Bounty:** Awarded, amount hidden. Severity High (8.7).
- **Lesson / pattern:** Trusting client-supplied attachment HTML plus a sanitizer that keeps behavior/URL data attributes plus framework nonce propagation turns a stored embed into same-origin RCE-equivalent JS and a persistent privilege grant. Rebuild attachment markup server-side, disable ALLOW_DATA_ATTR, and never copy the page nonce to cross-origin frame scripts.
- **Source:**
  - Title: Cross-account stored XSS on app.basecamp.com leads to persistent administrator takeover
  - Author: matty69v
  - URL: https://hackerone.com/reports/4175245
  - Date: 2026-11-13 (disclosed)
  - Disclosed report: https://hackerone.com/reports/4175245
  - Raw file: `sources/xss/04-basecamp-cross-account-stored-xss-admin.md`

Tags: stored, filter-bypass, account-takeover, admin-fire
