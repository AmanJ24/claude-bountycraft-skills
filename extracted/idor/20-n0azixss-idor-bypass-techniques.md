# IDOR bypass techniques: a phased testing methodology (guide)

- **Vuln type / sub-variant:** IDOR / BOLA, horizontal and vertical, with a catalogue of access-control bypass variations, batch operations, GraphQL and API-version angles. Guide, not a single finding.
- **Target type:** Web apps and APIs: profile/account settings, order history, invoices, document downloads, financial transactions, messages, admin panels, microservices.
- **Recon signal:** Object references appearing as query params, path segments, custom headers (user or account ID headers), cookies, or JSON/XML body fields; also sequential numbers, predictable UUID patterns and base64/hex-encoded IDs.
- **Root cause:** Authorization not enforced per object and per request; the server trusts client-supplied IDs, or checks only one request location or HTTP method, and some API versions or batch endpoints skip checks.
- **Exploitation steps:** (method taught)
  1. Inventory parameters: user identifiers (user_id, uid, account/customer IDs, username, email, phone), resource identifiers (order, invoice, document, file, message, comment, post IDs), file operation params (file, attachment, download, export), config IDs.
  2. Build a baseline: create your own resources, note IDs and URL shapes, then request own, adjacent, nearby and far-range IDs and compare responses; watch for inconsistent codes and informative errors.
  3. Distinguish horizontal (same role, other user) from vertical (regular user reaching admin objects); fill a test matrix of role vs own/other/admin ID with expected and actual result per cell.
  4. Apply bypass variations when the direct request is denied (see next field).
  5. Try batch endpoints (bulk update or bulk download taking arrays of IDs) for missing per-item checks.
  6. Try GraphQL: query by ID, nested queries, aliases, field-level access.
  7. Try other API versions (v1, v2, v3), since older ones are often more permissive.
  8. Finish with a checklist: boundary, empty, negative, special-character and encoded values, other HTTP methods, unauthenticated access, and privilege escalation.
- **Bypasses used:** HTTP method override (swap GET, POST, PUT, DELETE); parameter pollution (duplicate ID with different values, mixing id and user_id); encoding tweaks on the ID (trailing encoded slash, null byte, trailing dot); spoofing identity via headers (X-User-ID style, X-Forwarded-User, impersonation headers) and cookies (user_id, uid); Referer set to an admin origin; moving the same ID between path, query and JSON body; alternate route shapes for the same resource (view/ID vs ID/view); guessing filenames when IDs are replaced by names; older API versions; batch/bulk endpoints.
- **Impact:** Generic: access, modify or delete other users' data; admin panel access, full user data exposure and financial modification rated critical in its severity guide; sensitive personal data and account modification rated high; non-sensitive profile data medium; public data low.
- **Bounty:** n/a
- **Lesson / pattern:** A 403 on the obvious request is not the end; re-try the same object through other methods, locations, headers, versions and bulk endpoints, because authorization is often enforced in only one path.
- **Source:**
  - Title: IDOR & Bypass Techniques: A Practical Methodology for Security Testing
  - Author: N0aziXss
  - URL: https://medium.com/@N0aziXss/idor-bypass-techniques-a-practical-methodology-for-security-testing-47376bbf1ac2
  - Date: 2026-07-13
  - Disclosed report: none
  - Raw file: `sources/idor/20-n0azixss-idor-bypass-techniques.md`

Tags: methodology, bypass, api, enumeration, graphql, escalation
