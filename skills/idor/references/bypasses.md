# IDOR: id discovery and access-check bypass catalogue

Two problems to solve when a plain id swap does not immediately work: (A) you do not have the victim's id, or (B) the swap is rejected. This covers both. Validate everything with accounts you control; never read a real third party's data.

## A. Getting the victim's identifier

### A1. It is predictable (just compute it)
- **Sequential integers**: increment/decrement from your own id. Create two accounts and diff ids to confirm the stride (MTN ids 572/573, DoD ~320k). (notes 03, 10)
- **Hex / dword / timestamp**: same idea in the other base; epoch-based ids cluster by signup time.
- **UUID v1**: time + MAC based, predictable if you know roughly when the object was created.
- **Mongo ObjectID**: 4-byte timestamp + machine + pid + 3-byte counter; neighbors are computable from one sample. (note 25)
- **Hash of known data**: if an id looks like MD5/SHA1/SHA256, try `hash(email)`, `hash(username)`, `hash(id)`. (note 25)
- **GraphQL global ids**: `gid://app/Type/1000401` is just a wrapped integer; decrement it. (note 21)

### A2. It is "unguessable" (UUID v4): find the leak, do not brute force
- **Error-message disclosure**: submit an existing email to signup/reset/invite and read the id in the "already exists / already verified" error (note 12); read denied-response bodies for the object (notes 06, 08).
- **Web archives**: `echo https://target/ | waybackurls` and `gau`; grep for reset, invite, share, embedded-form, verification URLs. These carry ids directly, or JWTs whose payloads do. **Expired tokens still leak ids** (the token is dead for login, the id inside is still valid). (notes 16, 18)
- **Page source / JS**: object ids in `data-*` attributes, JS variables (`entityIds`), preloaded JSON. (note 09)
- **GraphQL**: list queries reveal sequential ids and gaps; introspect the schema (or rebuild it with Clairvoyance / error suggestions) to enumerate every id-taking operation. (note 23)
- **Second object you own**: for verification/OTP flows, create two resources you control and cross-wire their ids. (note 11)

## B. The swap is rejected (403/401/422): make the check miss

The check almost always guards one request shape or one code path. Try these, re-testing the same object each time.

### B0. Read the denied body first
Error serializers routinely include the object that was just refused. HackerOne `reference.json` returned 422 but embedded the report under `context.subject` ($15k); Pressable's validation error rendered the foreign app's client secret. A 403 is not a dead end until you have read its body. (notes 08, 06)

### B1. Change the HTTP method
Swap `GET` <-> `POST` <-> `PUT` <-> `PATCH` <-> `DELETE`. The ownership check may be wired to one verb while another reaches the same handler. (notes 20, 25)

### B2. Change content type / value shape
- XML -> JSON or form-encoded.
- Scalar -> array: `id=2` becomes `id[]=2`, or `{"id":2}` becomes `{"id":[2]}`. Resolvers often validate the scalar and act on the array (or vice versa).
- Wrap or nest the id differently than the client does. (notes 20, 25)

### B3. Parameter pollution / two ids
Send your id and the victim's together: `?id=self&id=victim`, or `id` in the path plus `user_id` in the body, or the same key twice with different values. The authorization check reads one occurrence; the data layer uses the other. (notes 20, 25)

### B4. Move the id between locations
Path vs query vs body vs header. A body `owner_id` can override the URL object the check validated (GitHub cross-repo: checked URL repo, acted on body `owner_id`). Try the id in each location, and in two locations at once. (note 21)

### B5. Spoof identity via headers / cookies
Add `X-User-Id`, `X-User`, `X-Forwarded-User`, impersonation or tenant headers; change an `uid`/`user_id`/`cam` cookie. Some back ends trust these over the session. (notes 16, 20)

### B6. Use a different endpoint for the same object
- **Side-effect action**: when the read path is blocked, a like/report/share/reply endpoint may return the object (VRoid hidden comment via the heart endpoint). (note 09)
- **Sibling route / alias**: `view/<id>` vs `<id>/view`, export vs view, a `.json` variant, a mobile/api host.

### B7. Older API version / batch endpoint
- `/api/v1/...` is often laxer than `/v2`. Try every version. (note 20)
- Bulk/batch endpoints taking **arrays of ids** frequently skip the per-item ownership check that the single-item endpoint enforces. (note 20)

### B8. Role and auth-state changes
Re-run the exact cross-object request as each role; the check may be enforced only on the low-priv path and missing on the admin path (cross-tenant delete worked as tenant admin, denied as member). Also try fully unauthenticated (unauth mass ATO). (notes 13, 10)

### B9. Wildcards and boundary values
Try `*`, `%`, `_`, `.`, negative, zero, very large, and empty ids to see if the backend returns all objects or a default. (note 25)

## Automation (authorized scope only)
Burp auth-testing extensions replay a captured request under another account's session automatically: Autorize, AuthMatrix, Authz. Use them to sweep many endpoints across two roles rather than hand-swapping each. Keep to your own test accounts and the program's rate limits. (note 25)
