# HackerOne blog: finding BAC in the wild (permissions mapping, Autorize)

- **Vuln type / sub-variant:** methodology/taxonomy (BAC vs IDOR, identifier types, systematic role x action testing)
- **Target type:** generic web apps and APIs with multiple roles
- **Recon signal:** object identifiers in URLs/bodies (numeric ids, UUIDs, hashes, usernames, composite keys); functions with a "from"/"owner" field in a request body; multiple roles with differing capabilities
- **Root cause:** access rules are custom per function or object, so any missed check is a hole; frameworks cannot enforce them by default
- **Exploitation steps:**
  1. Build a matrix: rows are roles (unauthenticated, editor, admin, etc.) x actions (create, read, update, delete).
  2. Fill "should have access" from business logic, then test each "no" cell and record "does have access"; any no/yes mismatch is a finding.
  3. Replay the whole browsing session as a lower-privileged user with Autorize (Burp) and compare response similarity to spot successes.
  4. Tamper with identifiers (increment numeric ids, swap account ids in transfer-style bodies) to reach data or functions of other users.
  5. Harvest UUIDs/hashes from normal app flows since guessing them is infeasible.
- **Bypasses used:** identifier taxonomy (user-chosen, natural keys, composite keys, numeric, UUID, hash); examples of non-IDOR BAC: forced browsing to /admin, claiming another username in a request, HTTP verb tampering (POST instead of GET)
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** IDOR is a subset of BAC; test functionality (not only data) and do it systematically with a role/action matrix plus automated cross-user replay.
- **Source:**
  - Title: How To Find Broken Access Control Vulnerabilities in the Wild
  - Author: Luke Stephens (Hakluke)
  - URL: https://www.hackerone.com/blog/how-find-broken-access-control-vulnerabilities-wild
  - Date: unknown (scraped 2026-11-17)
  - Disclosed report: none
  - Raw file: `sources/access-control/03-hackerone-blog-bac-in-the-wild.md`

Tags: taxonomy, methodology, reference, horizontal-privesc, vertical-privesc, method-based, data-exposure
