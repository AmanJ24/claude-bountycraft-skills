# Business logic reporting

Logic flaws are disputed more than any other class, because the request is valid and the vendor may call the behavior intended or a business decision. A good report removes that argument by naming the invariant, proving net benefit, and quantifying cost. Template and lessons below.

## Write-up template
1. **Title**: the invariant violated and the outcome. "Negative-quantity line item reduces order total below the real price"; "OTP not bound to resource id allows claiming any unclaimed listing".
2. **Summary**: one or two sentences, lead with the demonstrated advantage (paid $X for a $Y order; turned a $20k offer into ~$600k of fee-free processing; claimed a listing I do not own).
3. **The invariant and the missing enforcement**: state the rule the app relies on and exactly where the server fails to enforce it. "Server checks that the total matches the submitted lines but never validates quantities as positive or reprices from its catalog." "The verify endpoint checks the OTP/requestId pair but not that it was issued for this resId." "The role limit exists only in the UI; the API accepts any project member."
4. **Steps to reproduce**: numbered, with two accounts where relevant (attacker A, victim/second-tier B), the exact request(s), and the tampered field(s). Keep it to a 10-minute, from-scratch repro on test data.
5. **Proof of net benefit (end to end)**: show the real effect, not a changed field: the order fulfilled at the lower price, the entitlement used, the resource owned, the victim's data returned. Include the settlement/checkout confirmation, or a test-environment charge with test cards.
6. **Impact and scale**: map to confidentiality/integrity/availability; quantify per-unit cost times volume, accounts/resources affected, and whether it is repeatable/unlimited/cross-tenant.
7. **Remediation**: the server-side control that restores the invariant (recompute from catalog; validate ranges; bind tokens to user+resource+action; enforce role/plan on the endpoint; idempotency key; atomic check-then-act with a lock/unique constraint; per-feature caps).

## Lessons from this set (what landed, what got dismissed)
- **Prove fulfillment, not display.** The WooCommerce price-tamper was closed because the IPN check re-validated the amount after the order; a lower number on the PayPal page is not impact (note 04).
- **Prove net benefit.** The license report was Informative because the account ended up with fewer seats, not more; demonstrate an advantageous end state (note 18).
- **Verify server-side effect.** The seats-rounding report was almost dismissed as front-end only until the effect was confirmed through checkout on staging with test cards (note 15).
- **Quantify the cost.** Stripe landed $5,000 partly by stating the ~3% cost per discount and the ~$600k abused; dollar figures move severity (note 11).
- **Reopen with higher impact.** Streamlabs was first duplicated against a cosmetic report; showing a financial consequence (real reward coupons, stream keys) reopened it (note 13).
- **Pre-empt "intended" / "business decision".** Currency arbitrage (note 07) and the 2FA-collaborator scope (note 21) were ruled working-as-designed; for entitlement/policy classes, argue against the control's stated purpose and cite accepted precedent (note 14).
- **Provide a video when flows differ in triage.** Mobile reproduction that redirects to a website confused triage until a video PoC was supplied (note 16).
- **Expect duplicates on obvious vectors.** Plain amount tampering is heavily reported; lead with a non-obvious field or a larger impact (note 10).

## Redaction and safety
- Use test accounts and test data only; never claim a real third party's resource, drain a real balance, or complete a real underpriced order for goods.
- Redact tokens, signed URLs, session cookies, and any real victim data in the PoC.
- State what you did not do (did not withdraw, did not enumerate further); stopping at proof raises trust and severity.
