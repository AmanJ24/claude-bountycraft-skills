# OWASP Business Logic Security Cheat Sheet: defensive rules and the flaw patterns they imply

- **Vuln type / sub-variant:** methodology/taxonomy
- **Target type:** Any feature handling money, permissions, identity or multi-step state (checkout, coupons, referrals, KYC, resets, transfers)
- **Recon signal:** Features that dispense value, depend on step order, check-then-act without locking, or accept pricing/identity/role fields from the request.
- **Root cause:** Code does what it was told, but the rule the business needs (invariant) is enforced only by the UI or not at all.
- **Exploitation steps (testable patterns derived from the defenses):**
  1. Client-sent values: try altering price, subtotal, tax, discount amount, shipping cost, tax-exempt flag, user ID, tenant ID, role. Hidden and disabled fields count as inputs.
  2. Ownership: re-test authorization on every request, not only when the resource is first loaded.
  3. Workflow: call later-step endpoints directly, skip, repeat and reorder steps. Hidden "current step" fields or cookies are not enforcement. Look for completion endpoints that run without the prerequisite state, and half-finished states that never expire.
  4. Replays: re-run one-time actions (bonus, coupon, voucher, referral).
  5. Races: any check-then-act (balance debit, single-use coupon, limit counter, slot booking, unique registration) is worth parallel requests, including single-packet HTTP/2 bursts.
  6. Value-dispensing abuse: multi-accounting, self-referral, coupon stacking, free-trial resets via re-signup, missing per-feature rate limits.
  7. Semantic input checks: out-of-range quantities, dates, amounts, and valid fields that form invalid combinations (checkout before check-in, cross-customer transfers).
  8. Contextual authorization: self-approval, canceling after lock, changing email to one in use or without verifying it; same operation reachable via a weaker alternate endpoint (older API version, webhook, internal tool).
  9. Currency swap with amount unchanged is named as a price example.
- **Bypasses used:** n/a (defensive reference). Defenses: server-side recompute, server-side state machines, transactions/row locks/conditional updates/unique constraints, idempotency keys, per-feature caps and rate limits, audit logging and anomaly alerts, written invariants with tests.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** For each invariant ("coupon once per user", "balance never negative"), ask what enforces it; if the answer is the UI, there is a bug to find.
- **Source:**
  - Title: Business Logic Security Cheat Sheet
  - Author: OWASP Cheat Sheet Series
  - URL: https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/business-logic/12-owasp-business-logic-cheat-sheet.md`

Tags: methodology, taxonomy, reference, defense, workflow-bypass, state-machine, price-manipulation, coupon-stacking, unlimited-redemption, discount-abuse, free-services, privilege-logic, order-of-operations, client-trust, missing-server-validation, currency-confusion
