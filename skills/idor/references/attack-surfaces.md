# IDOR attack surfaces (with example findings)

Each surface: what it is, the recon signal, how the bug reads back, and the source notes in `../../extracted/idor/`. Use it to decide where to look and what to expect.

## Profile / account settings writes (email, phone, username)
The highest-value takeover primitive in the set. A self-service update carries the owner's id in the body; the server trusts it.
- Atavist (note 01): email-change POST carried a body `id`; swap to victim id, then forgot-password = zero-click ATO. Ids sequential, so mass.
- MTN (note 03): profile-update used the path id and let `username` (which equals the login email) be changed; swap id + change username + reset = ATO. The identity field was `username`, not `email`.
- DoD .edu (note 10): `/chkUser.aspx` took a `UID` form field and worked **unauthenticated**; one logged-out POST overwrote any of ~320k accounts' email. Test write endpoints logged out.
- **Recon:** any "edit profile / change email / change phone" request. Look for your id in the body/path. Check which field actually controls login and reset delivery.

## Account destroy / destructive actions
Server acts on the identity in the body, not the session.
- Mozilla `POST /v1/account/destroy` (note 05): deletes the account whose `email` is in the body; for SSO-only accounts the `authPW` proof is absent, so the victim's email is enough. Session-object misbinding.
- Cross-tenant file delete (note 13): a GraphQL delete checked the caller's role but not that the file's UUID belonged to the caller's tenant. Only the privileged (admin) path missed the check.
- **Recon:** delete/destroy/deactivate endpoints. Replay with another id/email in the body under your own session. Re-test under each role.

## List / search / export with a tenant or org id
Object-level reads at scale, not single records.
- HackerOne `/bugs.json` (note 02): `organization_id` in the search body was not checked against membership; set it to another program's id (one was on a public policy page) and page everything with `limit`/`page`, including drafts via extra `substates[]`. CVSS 9.3.
- **Recon:** list/search/report/export calls carrying an org/tenant/account id. Swap it; tune paging and status filters to widen results.

## API object endpoints with leaky error paths
The object comes back even when the action is "denied".
- Pressable (note 06): `application[id]` update moved any app into your account (write IDOR); then sending only `application[id]` + CSRF token hit a validation-error path that rendered the victim app's **client id and secret**. Strip parameters to force the error path.
- HackerOne `reference.json` (note 08, $15k): endpoint returned 422 "no access" but embedded the report under `context.subject`. Read the denied body.
- Lead API (note 12): `POST /api/leads` with an existing email returned that lead's id in an "already verified" error, turning email knowledge into id disclosure; then `PUT /api/lead/<id>/phone-number` from the attacker session.
- **Recon:** after a 403/422, always read the full body. Try minimal/invalid bodies to trigger error serializers.

## GraphQL operations by node id
GraphQL makes IDOR systematic: every operation takes an `id`.
- Shopify billing (aggregate, note 21): `BillingDocumentDownload` / `BillDetails` took an incremental `BillingInvoice` id; other merchants' email, address, invoice, card last-4 leaked. $5k.
- HackerOne certifications / Copilot (note 21): `CreateOrUpdateHackerCertification` and the unreleased `DestroyLlmConversation` lacked ownership checks (delete anyone's).
- Inactive embedded form (note 18, $2,500): `EmbeddedSubmissionPage(uuid)` still resolved **deactivated** forms; UUIDs harvested from Wayback. Revocation did not revoke access.
- PortSwigger GraphQL (note 23): list results reveal sequential ids with gaps; query the single-object op for the missing id to reach unlisted/private objects.
- **Recon:** proxy every GraphQL request, find each `id` variable, swap it; introspect the schema for every id-taking operation; watch JS bundles for new operations.

## Side-effect endpoints that echo the object
When the read path is locked, an action endpoint may return the object.
- VRoid hidden comments (note 09, $500): the like/heart endpoint `/api/statuses/<id>/hearts` returned a comment's body and author even after the owner disabled comments. Visibility was enforced only on the read path.
- **Recon:** for a hidden/forbidden object, try every other action by id (like, report, reply, share, pin) and inspect the response.

## OTP / verification keyed by resource id
The code is bound to the user but not the specific resource.
- OTP-IDOR (note 11): created two phone resources (UUIDs), requested an OTP for resource B, then submitted it against resource A's verify URL; server accepted it. Also a client-side-only region filter bypassed server-side.
- **Recon:** create two of the same resource you own, cross-wire their ids and tokens. Ask "what is this OTP actually bound to?"

## File / attachment / document / payment access
Often "unguessable" UUIDs as the only control, frequently cross-tenant.
- Credit-card API (note 16): `external_customer_id` (mirrored in a `cam` cookie) trusted without ownership check; card data read for other customers. Ids harvested from archived reset-link JWTs.
- Report/PDF export (note 14): enumerable job id -> download another user's exported PDF.
- **Recon:** downloads/exports/attachments by id or UUID. Swap across accounts; if unguessable, hunt the leak.

## ID discovery surfaces (feed the above)
- Archived URLs: `waybackurls`, `gau`, filter for reset/invite/share links; decode JWTs in them (expired tokens still leak ids) (notes 16, 18).
- Error messages that return the object's id (notes 08, 12).
- Page source / JS (`entityIds`), GraphQL list-id gaps, schema introspection (notes 09, 23).
- Predictable formats: sequential ints, UUID v1 (time-based), Mongo ObjectID (timestamp+machine+counter), hash of a known value (note 25).

## Parameter / reference name quick-list
`id, user_id, uid, userId, account_id, accountId, customer_id, external_customer_id, organization_id, org_id, tenant_id, company_id, application[id], app_id, report_id, reference, object_id, file_id, document_id, attachment_id, invoice_id, order_id, message_id, comment_id, status_id, lead_id, profile_id, email, username, owner_id, node id (GraphQL), uuid, guid, slug`
