# IDOR beyond id=123: reference formats, operations and hiding places (guide)

- **Vuln type / sub-variant:** General IDOR / broken object-level authorization (BOLA), methodology covering read, update, delete and action IDORs, plus role confusion. Guide, not a single finding.
- **Target type:** Any web app or API; examples cover orders, invoices, files, exports, user profiles, admin action endpoints, JSON APIs.
- **Recon signal:** Any value in a request that looks like a reference to an object: number, UUID, token, filename, slug, or an ID nested in JSON. IDs leaked in emails, JS variables, HTML comments, error messages and CSV/JSON exports are also leads.
- **Root cause:** Server loads or acts on whatever object the supplied reference points to, without confirming the object belongs to the caller or that the caller's role allows it. Composite references (user_id plus order_id) often get authorized on only one field.
- **Exploitation steps:** (method taught)
  1. Spot candidate references across all formats: sequential ints, sparse ints (1000, 2000, 3000), UUIDs, filenames, slugs, tokens, composite pairs.
  2. Tamper: go higher or lower, or swap in an ID observed elsewhere (email links, JS, another response, exports). A leaked non-guessed ID still counts.
  3. For composite refs, change only the object ID while leaving your own user ID, to see which field the backend really uses.
  4. For sparse IDs, probe around known values with stepping (plus or minus 10 or 100) and use error messages to learn which IDs exist.
  5. Test every CRUD operation, not just read: update (change another user's email or settings), delete (remove posts, files, tickets), and workflow actions (approve, confirm, cancel, ban) on objects you do not own.
  6. Inspect JSON bodies, including deeply nested IDs, and try path/body mismatch: own ID in URL with victim ID in body, and the reverse.
  7. Check file, download and export endpoints for other users' reports, backups, invoices.
  8. Check role confusion: low-privilege account calling admin-style endpoints or pointing admin-only objects.
- **Bypasses used:** none as such; the guide's tricks are reference-format variety, path/body mismatch, and using leaked IDs instead of guessing.
- **Impact:** Described generically: viewing others' profiles, orders, invoices, tickets; modifying emails or settings; deleting resources; triggering admin or business-workflow actions (usually high severity). No concrete incident.
- **Bounty:** n/a
- **Lesson / pattern:** Treat every identifier as a pointer and ask what happens when it points at someone else's object, for every operation, in every place IDs appear (URL, JSON, filenames, tokens).
- **Source:**
  - Title: IDOR Is Not Just id=123: A Simple Guide to Smarter IDOR Hunting
  - Author: Muhammed Asfan
  - URL: https://medium.com/@MuhammedAsfan/idor-is-not-just-id-123-a-simple-guide-to-smarter-idor-hunting-5f58de1109dd
  - Date: 2026-01-03
  - Disclosed report: none
  - Raw file: `sources/idor/19-asfan-idor-not-just-id123.md`

Tags: methodology, numeric-id, uuid, api, delete, escalation
