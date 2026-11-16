# IDOR escalation and impact

The same id swap can be worth $50 or $10,500. The difference is the rung you demonstrate. Climb as high as you safely can with your own test accounts, then stop. Rungs and examples are from the set (notes 21 and the individual findings).

## The escalation ladder

**Rung 1 - Read (data disclosure).** Most IDORs start here. Swap the id, get another account's data back.
- Credit-card API: `external_customer_id` swap returned card token, expiry, default flag; scaled to hundreds via archived-JWT id harvesting (note 16).
- HackerOne `/bugs.json`: private report metadata across all orgs, CVSS 9.3 (note 02).
- Even read-only is high-impact when the data is sensitive (payments, PII, private reports).

**Rung 2 - Update (modify victim data).** Escalate read to write.
- MTN / Atavist: change the victim's email/username (note 01, 03).
- Lead API: change any user's phone number given their email (note 12).
- Write turns a medium into a high.

**Rung 3 - Delete (destructive, usually rated higher, irreversible).**
- Mozilla `account/destroy`: delete any SSO-only account by email (note 05).
- Cross-tenant file delete, admin path only ($3k, note 13).
- HackerOne certifications / Copilot conversations: delete anyone's (note 21).

**Rung 4 - Account takeover.** The rung where bounties cross $5k.
- Change-email IDOR + public forgot-password = zero-click ATO (Atavist, MTN, DoD) (notes 01, 03, 10).
- Leaked credential reused: Pressable error path leaks client id+secret -> API takeover (note 06).
- Session reuse across sibling sites (Starbucks, note 21).

**Rung 5 - Mass / unauthenticated.** Top of the ladder, top bounties.
- Sequential ids = every account: loop the id (MTN ~500, DoD ~320k) (notes 03, 10).
- Unauthenticated write: no session needed at all (DoD `/chkUser.aspx`, note 10).
- Cross-tenant: one bug affects every tenant (rated higher for that reason) (notes 02, 13, 21).
- Aggregate cites PayPal $10,500 (all business accounts) and Shopify $5,000 (every merchant's billing) (note 21).

## Standard escalation chains
- **Change-email -> forgot-password -> login.** The most repeated takeover chain. Check which field actually controls reset delivery (sometimes `username`, not `email`). (notes 01, 03, 10)
- **Write IDOR that looks low -> minimal/invalid body -> error path leaks the object -> use leaked secret.** (note 06)
- **Read id from archive/JWT/error -> replay against the object endpoint -> repeat across harvested ids for scale.** (notes 16, 18)
- **Blocked read -> side-effect endpoint (like/report) echoes the object.** (note 09)
- **Denied with 422/403 -> the denied body contains the object.** (notes 08, 06)

## Severity signal (how the set was rated)
- Cross-tenant/private data read, or any takeover: critical/high (CVSS 9+): notes 02, 08, 10, 06.
- Single-user data read or targeted update: medium/high: notes 09, 12, 16.
- Needs the victim's credentials or heavy preconditions: downgraded (Mozilla's credentialed variant was low until the SSO-only, email-only variant raised it) (note 05).
- Not IDOR at all (client-value tampering): rated as business-logic, and only if downstream impact is shown (note 07).

## What raises the rating in practice
- Demonstrate the **highest rung reached**, not the first.
- Show the id is **reachable at scale** (sequential, or a working harvest path). This is what turned the embedded-form N/A into a valid $2,500 report (note 18).
- Prove **real impact** (returned PII, changed login, deleted object, secret used), not a UI change.
- Note the **blast radius** (one user vs all users vs all tenants).
