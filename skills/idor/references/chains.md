# IDOR chains and compositions

A standalone cross-tenant *read* IDOR is usually rated low-to-medium. The money is in chaining that read, or an equally cheap write, into a state-change the app never meant to allow cross-tenant: account takeover, financial movement, or privilege escalation. When you confirm one IDOR, immediately ask: **what state-change endpoint accepts the same id and might also be unchecked?** The second half is often the *same* missing-ownership bug class as the first, so it is easier to find than it looks.

Every chain below is grounded in findings from our set (note numbers refer to `../../extracted/idor/`); the composition framing is standard operator practice.

## Chain 1 - Email/username change -> forgot-password -> account takeover
The most repeated takeover chain in the set.
- **A.** Write-IDOR on the profile endpoint: `PUT/PATCH` with the victim's id in path or body changes their email or username without an ownership check (Atavist, MTN, DoD) (notes 01, 03, 10).
- **B.** Trigger the normal forgot-password flow for the victim. The reset link now goes to the attacker-controlled address.
- **C.** Open the link, set a password, log in as the victim. Zero-click, no victim credentials.
- **Watch:** confirm *which* field actually drives reset delivery. Sometimes it is `username`, not `email` (note 03). Changing the wrong field looks like it works but does not deliver the reset.
- **Scale:** sequential ids make this mass ATO (MTN ~500 accounts, DoD ~320k, unauthenticated) (notes 03, 10).

## Chain 2 - Read-IDOR -> leaked secret/token in the response -> API or account takeover
A read that looks low becomes critical because of *what* it returns.
- **A.** Read another object by id, or hit a minimal/invalid body that drops you onto an error path.
- **B.** The returned object, or the error serializer, includes a secret: Pressable's validation error renders the foreign application with its client id and secret (note 06); HackerOne `reference.json` 422 returns the refused report ($15k, note 08).
- **C.** Use the leaked secret against the API (OAuth client takeover, direct object access) or follow a token into a reset flow.
- **Lesson:** always read the denied/error body, and always scan a read-IDOR response for credentials, tokens, or ids that feed the next request.

## Chain 3 - Double-IDOR: list-by-id -> act-on-id
Two cheap bugs compose into direct impact.
- **A.** First IDOR lists the victim's objects by their id (orders, files, members), yielding legitimate child ids.
- **B.** Second IDOR acts on one of those child ids without checking ownership: delete, refund, modify.
- **Impact shape:** cross-tenant file delete (note 13); the read->delete and read->modify pairs are the common form. On commerce targets the act half is a refund/withdraw/transfer = direct financial loss.
- **Operator note:** the list half is often an easier find; use it to harvest the ids that make the act half demonstrable at scale.

## Chain 4 - IDOR membership-add + mass assignment -> privilege escalation on the victim tenant
- **A.** A horizontal IDOR adds the attacker to the victim's team/org (`POST .../{victim_org}/members`) with no ownership check.
- **B.** The body accepts an unfiltered field the API did not mean to expose (`role`, `permissions`, `is_admin`): mass assignment. Set yourself to owner/admin.
- **C.** Now you hold admin over the victim tenant and can remove the real owners.
- **Caveat from our set:** keep this distinct from pure client-value tampering, which is business-logic, not IDOR (Acronis price manipulation, note 07). The IDOR here is the cross-tenant membership write; mass assignment is the amplifier. Prove the role actually took effect server-side, not just in the UI.

## Chain 5 - Soft-delete / revoke IDOR -> stale token still valid -> persistent access
- **A.** The "remove member" or "revoke" endpoint flips an `active=false` flag but does not invalidate the existing session or API token.
- **B.** Capture a session/PAT while still a member, have access revoked through the normal flow, then keep calling the API with the captured credential.
- **Impact:** temporal IDOR, post-termination cross-tenant access for as long as the token lives. This is the "keeps working after the object is revoked/hidden/deleted" signal from the skill's Recognize section, turned into a chain.

## Chain 6 - Read-IDOR -> downstream XSS/cookie theft (cross-skill)
When a read/download IDOR feeds a field that is later rendered or set in a response header, the chain crosses into XSS.
- **A.** IDOR on a file-download or object-read endpoint.
- **B.** An attacker-controlled field on that object (filename, title) is reflected unsanitized into HTML or a `Content-Disposition` header on fetch.
- **C.** Victim opens the object, injected script runs same-origin, steals session.
- **Cross-reference:** see the `xss` skill for the response-context injection half. Neither primitive alone is critical; the composition is.

## The operator pattern
After confirming a read-IDOR at A, look for the matching state-change that shares the id:
1. password reset / email change at the terminal step -> ATO (Chain 1)
2. refund / withdraw / transfer / delete -> financial or destructive (Chain 3)
3. role-change / membership-add -> privilege escalation (Chain 4)
If your read-IDOR composes to none of these, the standalone rating is what you get. Hunt for both halves anyway; the second usually shares the first's auth-check gap.
