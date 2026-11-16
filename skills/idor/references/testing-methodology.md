# IDOR testing methodology (step by step)

Research/methodology guidance. Against a real target, only proceed under an authorized, in-scope program, with accounts you control. Never read or change a real third party's data: that is unauthorized access, not a PoC. Use test accounts and stop at proof (see `../../CLAUDE.md`).

## Step 0 - Set up two accounts you control
This is the core instrument. Create account A (attacker) and account B (victim). For multi-tenant apps, two tenants/orgs. For role bugs, the same object reachable by an admin and a non-admin. Keep A and B in separate browser containers/profiles so sessions do not bleed. Now you can read B's real identifiers legitimately and prove cross-account access without touching a stranger.

## Step 1 - Inventory every object reference
Browse both accounts through a proxy. For every request, list identifiers in: URL path, query string, JSON/XML body, custom headers (`X-User-Id` style), and cookies (`uid`, `cam`, etc.). For each, record the **format**: sequential integer, hex, epoch timestamp, UUID v1 vs v4, Mongo ObjectID, base64, hash, slug. Format tells you how discoverable the id is (see `bypasses.md`).

## Step 2 - Run the five questions per endpoint
From the 252-report analysis (note 21):
1. Does this endpoint accept an object identifier? If yes, proceed.
2. Is the identifier sequential or predictable? Increment, decrement, try B's id.
3. Does the action go beyond read? Test update and delete, not just read.
4. Does the server bind the session to the object, or will it act on a body/path id you swap while keeping your own cookie?
5. Is this a new feature? Check JS bundles and GraphQL schema/introspection for recently added operations (least review history).

## Step 3 - Swap A's id for B's and compare
Replace A's identifier with B's real id (you know it from Step 0) and send from A's session. Compare status and body against the legitimate request.
- Different data returned = Rung 1 (read). Record it.
- Then repeat for **update** and **delete** actions on B's object.
- For list/search endpoints, swap the tenant/org id and tune paging (`limit`, `page`) and status filters to widen results.

## Step 4 - If denied, do not stop
A 403/401/422 means a check guards *this shape*. Before concluding "secure":
- **Read the full denied body** (it often contains the object: notes 06, 08).
- Work the bypass ladder in `bypasses.md`: method swap, content-type/shape change, array-wrap the id, parameter pollution, move id path<->body<->header, spoof id headers, sibling/side-effect endpoint, older API version, batch endpoint.
- Re-test under **each role** and **unauthenticated** (notes 10, 13).

## Step 5 - If the id is "unguessable," hunt the leak
Do not brute-force a UUID v4. Find where it is disclosed:
- Signup/reset/invite endpoints that echo the id in errors (note 12).
- Archived URLs: `echo <domain> | waybackurls` / `gau`, grep for reset/invite/share/embedded paths; decode JWTs in them, expired tokens still carry ids (notes 16, 18).
- Page source / JS variables (`entityIds`), GraphQL list results (query the gaps), schema introspection or suggestions (Clairvoyance) (notes 09, 23).
- Predictable generators: UUID v1 (time-based), Mongo ObjectID (timestamp+machine+pid+counter) (note 25).

## Step 6 - Build a role x object matrix (for access-control depth)
For vertical/cross-tenant work, fill a grid: rows = role (anon, user A, user B, admin, other-tenant admin), columns = own id / other user's id / admin-only id. Mark expected vs actual per cell. The gaps are the bugs, and they often appear only in the privileged path.

## Step 7 - Escalate, then stop at proof
Push to the highest rung you can with your test accounts (read -> update -> delete -> account takeover -> mass/unauth; see `escalation-and-impact.md`). Capture minimal evidence: B's identifier, the response showing B's data or the changed state, one screenshot. Do not enumerate other users or move laterally. For a real program, stop before touching anyone outside your test accounts and say so in the report.

## Tools seen in the sources
- Burp (intercept/repeater) and auth-testing extensions: Autorize, AuthMatrix, Authz (replay a request across roles automatically) (note 25).
- `waybackurls`, `gau` for archived id harvesting; `jwt` decoders (notes 16, 18).
- Clairvoyance for GraphQL schema recovery when introspection is off (note 23).

## Pitfalls / false-positive guards
- Confirm the object belongs to a **different** account before claiming IDOR (that is what account B is for).
- Client-supplied price/quantity/role is business-logic/mass-assignment, not IDOR (note 07).
- An active, intentionally-public reference is not a bug; a revoked/inactive/other-tenant one is (note 18).
- A random UUID that is actually ownership-checked is not IDOR; you need a failed check or a leak.
