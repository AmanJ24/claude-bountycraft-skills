# SMB-based SSRF in SVG emblem editor leaks server NTLMv2 hash (Rockstar Games)

Source completeness: partial (report body not public; only team summary and researcher summary)

- **Vuln type / sub-variant:** SSRF via alternate protocol (SMB/UNC path) through server-side SVG rendering. Outbound authentication, not an HTTP read.
- **Target type:** Image/SVG upload processed by a rendering library (emblem editor).
- **Recon signal:** Not stated in source beyond the fact that the emblem editor accepts user-submitted SVG files that the server renders.
- **Root cause:** The server-side SVG processing (ImageMagick with its SVG renderer) resolved UNC paths referenced in the SVG and attempted to authenticate to the remote host. User-submitted SVGs were not sanitized for such references.
- **Exploitation steps:**
  1. Craft an SVG that references a UNC path pointing to an attacker-run SMB listener (exact markup not in source).
  2. Upload it through the emblem editor.
  3. The server connects to the attacker's SMB host and sends the process account's login and NTLMv2 hash.
- **Bypasses used:** alternate protocol: SMB/UNC (no filter evasion described).
- **Impact:** Disclosure of the service account's NTLMv2 hash, which could be cracked offline or used in an SMB relay attack; the reporter says it could lead to RCE. The "domain credentials" claim in the title is from the original title; the summaries do not detail cracking or relay success.
- **Bounty:** $1,500
- **Lesson / pattern:** When a Windows-hosted image pipeline renders user SVG, test non-HTTP references (UNC/SMB) and watch for inbound auth to your listener, not just HTTP callbacks.
- **Source:**
  - Title: SMB SSRF in emblem editor exposes taketwo domain credentials, may lead to RCE
  - Author: alexbirsan
  - URL: https://hackerone.com/reports/288353
  - Date: 2018-01-12 (disclosed; reported 2017-11-08)
  - Disclosed report: https://hackerone.com/reports/288353
  - Raw file: `sources/ssrf/10-h1-288353-rockstar-emblem-editor-smb-ssrf.md`

Tags: svg-upload, imagemagick, smb, ntlm-hash, alternate-protocol, blind
