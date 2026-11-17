# Race on fee-discount acceptance endpoint gives unlimited fee-free processing (Stripe)

- **Vuln type / sub-variant:** Single-use offer redeemable many times (missing idempotency/state check, then a race condition after a partial fix)
- **Target type:** Dashboard AJAX endpoint that accepts a promotional fee discount offer
- **Recon signal:** The researcher was a real customer with a $20,000 fee discount offer pending acceptance. A one-shot "accept offer" action is a classic candidate for double-redeem or race testing.
- **Root cause:** The accept endpoint did not enforce that an offer (identified by an fdo_ ID) could be accepted only once. Each call applied a fresh discount. The first fix added an already-accepted check, but it was not atomic, so concurrent requests still passed the check.
- **Exploitation steps:**
  1. Intercept the accept-offer request from the dashboard for the offered discount.
  2. Replay it 30 times. Even plain repeated calls applied 30 discounts, turning a $20k offer into about $600k of fee-free processing.
  3. After the first fix (error "already accepted" appears), use Turbo Intruder with a gated single-packet style release across 30 connections.
  4. About 23 of 30 still succeeded, proving a remaining race. A second fix stopped this.
- **Bypasses used:** Concurrent requests released together via a gate to beat the non-atomic already-accepted check.
- **Impact:** Unlimited fee discounts. Estimated cost to Stripe about 3% of each discount value (about $600 per $20k discount abuse).
- **Bounty:** $5,000 plus two $50 retest awards.
- **Lesson / pattern:** For any redeem/accept/claim action, test plain replay first, then race it. After a fix, always retest with concurrency, since "check then act" patches often stay racy.
- **Source:**
  - Title: Fee discounts can be redeemed many times, resulting in unlimited fee-free transactions (Stripe)
  - Author: ian
  - URL: https://hackerone.com/reports/1849626
  - Date: 2023-01-28
  - Disclosed report: https://hackerone.com/reports/1849626
  - Raw file: `sources/business-logic/11-h1-1849626-stripe-unlimited-discount.md`

Tags: discount-abuse, unlimited-redemption, state-machine, order-of-operations, missing-server-validation, financial-loss
