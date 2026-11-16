# PayloadsAllTheThings: IDOR ID types and tricks (reference)

- **Vuln type / sub-variant:** IDOR with a focus on identifier formats (numeric, guessable names, weak PRNG-based, hashed) plus request-level tricks to get past access checks.
- **Target type:** Any web or API endpoint taking a user, resource or file identifier (profile pages, user APIs, campaigns, messages, orders, tokens).
- **Recon signal:** The ID format itself tells you how predictable it is: decimal or hex counters, epoch timestamps, usernames or emails, base64-encoded values, UUID v1, MongoDB ObjectIDs, or hash of a known value.
- **Root cause:** Missing ownership check combined with identifiers that can be guessed, derived or computed.
- **Exploitation steps:** (method taught)
  1. Numeric: increment and decrement decimal, hexadecimal and Unix-timestamp IDs.
  2. Common identifiers: try names, emails, and base64-encoded forms of them.
  3. Weak PRNG: UUID/GUID v1 is predictable if creation time is known; MongoDB ObjectIDs are built from timestamp, machine ID, process ID and counter, so neighbors can be computed.
  4. Hashed parameters: if an ID looks like MD5, SHA1 or SHA2 of something guessable (username, email), compute the hash for the victim.
  5. Wildcard: send *, %, . or _ in place of the ID to see if the backend returns all users.
  6. Use authorization-testing Burp extensions listed (Authz, AuthMatrix, Autorize) to replay requests across roles.
- **Bypasses used:** Change HTTP method (for example POST to PUT); change content type (XML to JSON); turn a scalar into an array or wrap it in JSON array form; HTTP parameter pollution with your ID and victim ID both present under the same name; wildcard characters instead of IDs; encoding guessable values (base64, hashes). The page does not explicitly list path-versus-body swaps or second-copy ID injection beyond the pollution tip.
- **Impact:** Not detailed; linked examples include viewing another user's order info, deleting messages, deleting campaigns and reading another user's token (HackerOne reports #287789, #697412, #1969141, #1464168), plus a MongoDB ObjectID prediction writeup.
- **Bounty:** n/a
- **Lesson / pattern:** Identify how an ID was generated; if it is derived from time, counters or hashes of known data, it is computable, and if access checks resist, vary method, content type, shape and duplicates.
- **Source:**
  - Title: Insecure Direct Object References (PayloadsAllTheThings)
  - Author: swisskyrepo
  - URL: https://swisskyrepo.github.io/PayloadsAllTheThings/Insecure%20Direct%20Object%20References/
  - Date: not stated in source
  - Disclosed report: links H1 #287789, #697412, #1969141, #1464168 in-page
  - Raw file: `sources/idor/25-payloadsallthethings-idor.md`

Tags: bypass, numeric-id, uuid, enumeration, methodology, api
