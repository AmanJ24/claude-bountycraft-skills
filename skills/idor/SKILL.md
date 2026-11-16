---
name: idor
description: Use when testing any feature that references a user or object by an identifier you can change (profile/email/phone updates, account delete, list/search with an org or account id, API objects, GraphQL operations, file/attachment/export downloads, like/report/share actions, OTP verification). Covers recognizing IDOR/BOLA, finding the object reference, swapping it across accounts, bypassing access checks, discovering IDs, avoiding false positives, and writing up impact. Research/methodology only: it does not send requests to live targets.
---

# IDOR / Broken Object-Level Authorization (BOLA)

Synthesized from 25 real bug bounty findings and references (HackerOne disclosed reports incl. a $15k HackerOne-platform report, a 252-report analysis, PortSwigger, PayloadsAllTheThings, practitioner writeups). Every pattern traces to `sources.md`. Bounties in the set ran from $50 to $15,000 (and the aggregate cites a $10,500 PayPal IDOR). The biggest payouts came from account takeover, mass/unauthenticated takeover, and cross-tenant data exposure.

**The one idea:** IDOR is never about *finding* the ID. It is about finding where the server resolves an object from a client-supplied reference and forgets to check that *this* requester owns *that* object. Guessing or leaking the ID is the easy half.

**Scope rule:** methodology and guidance only. It does not fire requests at live targets, and real testing needs two accounts you control and an authorized, in-scope program (see `../../CLAUDE.md`). Reading or changing another real user's data without authorization is a crime, not a PoC: use test accounts and stop at proof.

## 1. Recognize it

Every IDOR sits at the intersection of an **object reference** (id, uuid, slug, GraphQL node id, filename, email), an **action** (read / update / delete), and an **authorization check** that is missing, wrong, or bypassable. The signal is a request where the thing identifying *whose* object this is comes from the client, not from the session.

Signals, roughly in order of how often they paid out in this set:
- A request carries a **user/object identifier the client controls**: `id`, `user_id`, `uid`, `account_id`, `organization_id`, `customer_id`, `external_customer_id`, `application[id]`, `report_id`, a UUID in the path, a GraphQL `id` variable, or an `email`/`username` in the body separate from the session.
- A **write to your own profile** (change email, phone, username) that includes your id in the body. Classic takeover primitive (Atavist, MTN, DoD).
- A **list/search/export endpoint** that takes a tenant or org id (HackerOne `/bugs.json` `organization_id`).
- A **GraphQL operation** that takes an `id` (Shopify billing, HackerOne certifications, LLM conversations).
- A **"secret" UUID** treated as the only protection (file stores, report links, embedded forms).
- A **destructive endpoint** (`account/destroy`, delete file/photo/comment) that names its target in the body.
- An **id that doubles as identity** (username == login email), or an endpoint that keeps working **after the object is revoked/hidden/deleted**.

## 2. Where it hides

Feature classes that actually produced findings. Full list with example reports in `references/attack-surfaces.md`.

- **Profile / account settings writes** (change email/phone/username): id in the body, chained to forgot-password for takeover (Atavist $, MTN, DoD unauth mass ATO).
- **Account destroy / destructive actions**: server acts on the body's email/id, not the session (Mozilla `account/destroy`; cross-tenant file delete $3k).
- **List / search / export with a tenant or org id** (HackerOne `/bugs.json`; report/PDF exports).
- **API object endpoints** returning or moving objects by id, where an **error path leaks the object** (Pressable `application[id]` leaks client secret; HackerOne `reference.json` 422 body leaks report, $15k; lead signup error leaks lead id).
- **GraphQL operations** by node id, including unlisted objects found via id gaps, and unreleased operations found in JS bundles (Shopify, HackerOne certifications/Copilot, GitLab ML registry).
- **Side-effect endpoints that echo the object**: like/heart/report/share returning a hidden object's content (VRoid hidden comments).
- **OTP / verification** keyed by a resource id, where the code is bound to the user but not the specific resource (OTP-IDOR).
- **File / attachment / document access by id or "unguessable" UUID**, often cross-tenant (credit-card API; file stores).
- **ID discovery surfaces**: archived reset/invite URLs, JWTs in them, error messages, page source (`entityIds`), predictable ObjectIDs/UUIDv1.

## 3. Test methodology

Short version. Full step-by-step in `references/testing-methodology.md`.

1. **Get two accounts you control** (A attacker, B victim). Where relevant, two tenants/orgs and two roles. This is the core tool: you can read B's real ids legitimately and prove cross-account access without touching a stranger's data.
2. **Inventory every object reference** in every request: path, query, body (JSON/XML), headers, cookies. Note the id format (sequential int, hex, timestamp, UUID v1/v4, Mongo ObjectID, base64, hash).
3. **Run the 5 questions per endpoint** (from the 252-report analysis): (a) does it take an object id? (b) is the id predictable? (c) does the action go beyond read, test update and delete? (d) does the server bind the session to the object, or will it act on a body/path id you swap? (e) is this a new feature (check JS bundles and GraphQL schema)?
4. **Swap A's id for B's** and compare responses. Different data = Rung 1. Then test update and delete, not just read.
5. **If the direct swap is denied (403/422),** do not stop. Read the denied response body (it often contains the object), and work the bypass ladder in `references/bypasses.md` (method swap, content-type, array/JSON wrapping, parameter pollution, path-vs-body, id-in-two-places, header spoofing, older API version, batch endpoints).
6. **If the id is "unguessable,"** hunt the leak instead of guessing: signup/reset/invite errors, archived URLs (`waybackurls`, `gau`), JWT payloads, page source, GraphQL list-id gaps, ObjectID/UUIDv1 prediction.
7. **Test each role and the unauthenticated case.** Authorization bugs often live only in the privileged path (cross-tenant delete worked as admin, not as member) or vanish only when logged out matters (unauth mass ATO).
8. **Escalate impact, then stop at proof.** Push to the highest rung you safely can (read -> update -> delete -> takeover -> mass) using your own test accounts, capture minimal evidence, and report.

## 4. Known access-check bypass patterns

When the obvious id swap returns 403/401/422, the check usually guards one request shape or one code path. Summary; full catalogue in `references/bypasses.md`.

- **Read the denied body.** Error serializers frequently include the refused object (HackerOne `reference.json` 422; Pressable validation error renders the foreign app with its secret).
- **Change the HTTP method** (GET<->POST<->PUT<->DELETE): the check may exist on one verb only.
- **Change content type / shape**: XML to JSON, scalar to array (`id=2` -> `id[]=2` or `{"id":[2]}`), wrap in JSON. Resolvers often mishandle the alternate shape.
- **Parameter pollution / two ids**: send your id and the victim's both (`id=self&id=victim`, or `id` in path plus `user_id` in body). The check reads one, the action uses the other.
- **Move the id between locations**: path vs query vs body vs header. A body-supplied `owner_id` can override the URL object the check validated (GitHub cross-repo).
- **Spoof identity via headers/cookies**: `X-User-Id`, `X-Forwarded-User`, impersonation headers, `uid` cookie.
- **Use a different endpoint for the same object**: a side-effect action (like/report/share) or a sibling route that echoes the object when the read path is blocked (VRoid).
- **Older API version / batch endpoint**: `v1` is often laxer than `v2`; bulk operations taking arrays of ids skip per-item checks.
- **GraphQL**: query the single-object operation for ids missing from a list (gaps), enumerate operations via introspection or suggestions (Clairvoyance), pack guesses with aliases.

## 5. Common false positives

- **Trusted-client value tampering is not IDOR.** A client-supplied price, quantity, discount, or role flag is a **mass-assignment / business-logic** issue (Acronis "price manipulation"). Report it as that, and prove the downstream effect (gateway amount), not just a changed UI value.
- **Accessing your own data** via a different route. Confirm the object belongs to a *different* account (that is why you keep two accounts).
- **Public-by-design references.** An active, intentionally-shareable link (an embedded submission form while active, a public profile) is not a bug. The finding is reaching it when it should be private (revoked/inactive/other-tenant).
- **Properly-checked unguessable ids.** A random UUID that *is* authorization-checked is not IDOR just because it is in the URL. You need either an ownership-check failure or an id leak.
- **Same-tenant, same-permission reads** that the role is meant to have.
- **A 403 with an empty body and no other shape working.** Note it, but without data back or a working bypass it is often informational.
- **Self-inflicted "leak"** where the id came from your own session/data and no other account is affected.

## 6. Reporting

What made reports in this set land (template in `references/reporting.md`):
- **Two-account PoC.** Show account A reading/modifying account B's object, with B's identifier and the response. Use test accounts; redact any real victim data.
- **Climb the escalation ladder and demonstrate the top rung you reached:** read -> update -> delete -> account takeover -> mass/unauth. Deletes and takeovers rate higher; sequential ids plus takeover = mass impact = top bounties.
- **Name the exact missing check**: "server selects the record by body `id` and never compares it to the session user", "authorizes on the URL repo but acts on body `owner_id`", "error serializer returns the object it just refused".
- **Show the id is reachable**: sequential/predictable, or leaked (archive, JWT, error, schema gap). A real attacker path turns a "needs the id" N/A into a valid report (embedded-form case).
- **Prove real impact, not UI**: the returned PII, the changed login email, the deleted object, the leaked secret used against the API.
- **Stop at proof** and say so; several high ratings came with the researcher explicitly not enumerating further.

## 7. Sources

See `sources.md` (title, author, URL, disclosed report, bounty) and `../../extracted/idor/` for the per-writeup notes. Deeper references:
- `references/attack-surfaces.md`: feature classes with example reports
- `references/testing-methodology.md`: step-by-step, two-account workflow, the 5 questions
- `references/bypasses.md`: id discovery + access-check bypass catalogue
- `references/escalation-and-impact.md`: the read->mass ladder with examples
- `references/reporting.md`: write-up template
