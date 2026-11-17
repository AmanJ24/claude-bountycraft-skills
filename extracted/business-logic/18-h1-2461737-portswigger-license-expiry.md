# Duplicate payment notification corrupts license state, extending expiry of an existing license

- **Vuln type / sub-variant:** Business logic / license-logic flaw (state corruption when a second license purchase is processed against an existing license)
- **Target type:** Commercial software licensing and billing portal (renewals, per-user seat licenses)
- **Recon signal:** After buying an extra 5-year license for a company that already held a 4-user license, the reporter revisited the licenses page and saw the old license's seat count shrink while its expiry date jumped forward.
- **Root cause:** Purchase handling merged or overwrote the existing license record instead of treating the new order as independent. Per the vendor, the payment notification fired twice, so the account was downgraded (seats reduced) while the expiry date was inherited from the newer order. Idempotency of payment callbacks and per-license isolation were missing.
- **Exploitation steps:**
  1. Hold an existing license (4 seats, near-term expiry).
  2. Purchase one more license (1 seat, 5 years) and pay.
  3. Re-open the licenses page and compare: the old license shows fewer seats but the far-future expiry.
  4. Hypothesized abuse: buy a short license, then a long one, so both end up with the long expiry. Reporter proposed confirming on his own account with 1 seat for 1 year then 1 seat for 2 years, with a refund.
- **Bypasses used:** None. Passive observation of purchase side effects; reproduction was admitted to be hard and unconfirmed as exploitable.
- **Impact:** Not proven as an advantage. Vendor found the account ended up worse off (downgraded to 2 seats instead of 4) and treated it as a non-vulnerability bug; they promised a manual temporary license fix and invited replication steps that leave the user better off.
- **Bounty:** None (Informative; not stated).
- **Lesson / pattern:** Test purchase, renewal, and upgrade flows for cross-license side effects, and look at double-processed payment callbacks. A logic bug only counts when it nets the attacker a benefit, so demonstrate an advantageous end state, not just a changed field.
- **Source:**
  - Title: Incorrect logic when buy one more license which may lead to extend the expire date of existing license
  - Author: liru
  - URL: https://hackerone.com/reports/2461737
  - Date: 2024-04-13
  - Disclosed report: https://hackerone.com/reports/2461737
  - Raw file: `sources/business-logic/18-h1-2461737-portswigger-license-expiry.md`

Tags: license-logic, subscription-bypass, state-machine, assumption-flaw, order-of-operations
