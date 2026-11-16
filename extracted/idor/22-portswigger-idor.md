# PortSwigger Web Security Academy: IDOR concept page (reference)

- **Vuln type / sub-variant:** IDOR as an access-control flaw where user-supplied input directly selects an object; mostly horizontal privilege escalation, sometimes vertical. Two variants: direct database record reference and direct static file reference.
- **Target type:** Account pages keyed by customer number; static files such as stored chat transcripts with incrementing filenames.
- **Recon signal:** A parameter that is used as a record index (e.g. a customer number in a query string), or a static file URL whose name is a counter.
- **Root cause:** The reference is used directly in a back-end lookup or filesystem path with no additional authorization control. The page notes IDOR is only one example among many access-control implementation mistakes.
- **Exploitation steps:** (method taught)
  1. Find a request where a parameter or filename identifies an object.
  2. Modify the value (change the customer number, or the incrementing file name) to another user's.
  3. Confirm other users' records or files are returned, possibly exposing credentials or sensitive data.
  4. Once inside another account page, consider further moves: altering the user to a more privileged one (vertical), exploiting leaked passwords, or editing parameters.
- **Bypasses used:** none
- **Impact:** Horizontal data access to other customers' records or transcripts; potential vertical escalation and credential exposure. Links to an apprentice lab for practice.
- **Bounty:** n/a
- **Lesson / pattern:** Static files with sequential names are IDOR surface too, not just database-backed parameters.
- **Source:**
  - Title: Insecure direct object references (IDOR)
  - Author: PortSwigger
  - URL: https://portswigger.net/web-security/access-control/idor
  - Date: not stated in source
  - Disclosed report: none
  - Raw file: `sources/idor/22-portswigger-idor.md`

Tags: methodology, numeric-id, enumeration, pii-disclosure
