# Chained XSS in PDF converter to SSRF against headless Chrome remote debugging port

- **Vuln type / sub-variant:** HTML/script injection into a server-side PDF renderer (XSS to SSRF), reaching a local Chrome DevTools remote debugging endpoint. This is a CTF-style H1 event target, so the chain is long and artificial.
- **Target type:** Document-to-PDF converter running headless Chrome with remote debugging enabled, fed by a user display name.
- **Recon signal:** The user's name field is rendered into the converted PDF. The converter was stripping nothing at the point where an IDOR let the attacker edit another user's name, and the hunter noted that the renderer is a real browser that runs scripts.
- **Root cause:** Several flaws stacked: an account takeover through trailing angle-bracket characters that get stripped when the recovery QR is made; a CSP that allowed a whole third-party path on a code host, bypassable by path backtracking to load attacker JS from an arbitrary repo; an IDOR in the support review endpoint allowing the name of any user to be changed with angle brackets intact; and finally a PDF converter that executes injected script in a headless Chrome whose debugging port is reachable from inside.
- **Exploitation steps:**
  1. Register an email variant with trailing angle brackets so the recovery code issued matches the victim's address after stripping, taking over the victim account.
  2. Use stored XSS in the support chat, defeating the CSP by loading a script from the allowed code-hosting domain via path traversal into the attacker's own repository.
  3. Use the IDOR on the support review update call to set the victim's display name to a script tag with attacker JS.
  4. When the PDF converter renders the name, the script runs inside headless Chrome, and from there queries the local remote-debugging endpoint to list open tabs and their data, leaking the sensitive document (the flag).
- **Bypasses used:** CSP allowlist abuse via path backtracking on a permitted host; character-stripping normalization abuse for account takeover; HTML injection into renderer. No network filter bypass.
- **Impact:** Leak of other users' sensitive data via DevTools tab listing. Rated critical (CTF context).
- **Bounty:** not stated in source.
- **Lesson / pattern:** If a PDF or screenshot generator is a real headless Chrome, injected script can reach localhost services such as the debugging port, which exposes every open page; always check for local debug or admin ports from the renderer's viewpoint.
- **Source:**
  - Title: [h1-415 2020] SSRF in a headless chrome with remote debugging leads to sensible information leak
  - Author: d1r3wolf
  - URL: https://hackerone.com/reports/781295
  - Date: 2020-02-04
  - Disclosed report: https://hackerone.com/reports/781295
  - Raw file: `sources/ssrf/19-h1-781295-headless-chrome-remote-debug-h1ctf.md`

Tags: pdf-generator, headless-browser, xss-to-ssrf, devtools-port, chained-ssrf, ctf
