# SSRF eval run sheet (live, user-run) - 2026-11-16

Goal: follow ONLY what the skill tells you and see if each lab solves. For each lab, the
"skill path" below is derived from skills/ssrf/SKILL.md + references, not the PortSwigger
solution. Mark pass if the skill's guidance got you there; note any gap where you had to
go beyond the skill. All five are authorized self-owned training labs.

Tip: run through Burp; "delete carlos" / access admin is the lab's success condition.

## 1. Basic SSRF against the local server
Skill path: SKILL section 1 flags `stockApi` as a server-side fetch param (URL in a param).
Section 3 says expect full read. Set stockApi to `http://localhost/admin`, read the
response, then call the admin delete URL for the user.
Pass if: skill made you (a) spot stockApi as the sink, (b) swap to localhost, (c) read the body.

## 2. Basic SSRF against another back-end system
Skill path: exploitation-and-escalation.md "internal port scan / enumeration" + testing step 4.
Iterate stockApi over `http://192.168.0.1:8080/admin` ... `.255`, diff responses to find the
host, then delete the user on that host.
Pass if: skill directed internal-range enumeration and using response diffs to locate the host.

## 3. SSRF with blacklist-based input filter
Skill path: bypasses.md section 0 (rejection = blocklist on text) + section 3 (IP encodings).
Try `http://127.1/admin`, dword `http://2130706433/admin`, or obfuscate `localhost`
(double URL-encode, mixed case). One of these matches the lab's intended class.
Pass if: skill told you it's a text blocklist and to try IP-notation/obfuscation variants.

## 4. SSRF with filter bypass via open redirection
Skill path: bypasses.md section 0 (allowlist) + section 1 (redirects). The stockApi only
allows the site's own host; use the on-site open redirect (the `path`/`next` param on a
product's stock-check path) to 3xx the server-side fetch to `http://192.168.0.12:8080/admin`.
Pass if: skill pointed you to an allowlist + open-redirect-on-allowed-domain chain.

## 5. Blind SSRF with out-of-band detection
Skill path: SKILL section 2 (Referer is a hidden fetch surface) + testing step 2 (one OOB
callback) + section 3 (blind). Put your Collaborator/interactsh URL in the Referer header;
a server-side DNS/HTTP hit confirms blind SSRF. (Skill notes a lone DNS lookup still counts.)
Pass if: skill made you test the Referer header with an OOB host and frame it as blind.

## Results (fill in)
| Case | Pass/Fail | Skill gap noted? |
|------|-----------|------------------|
| 1 |  |  |
| 2 |  |  |
| 3 |  |  |
| 4 |  |  |
| 5 |  |  |
