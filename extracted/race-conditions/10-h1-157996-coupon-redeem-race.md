# Instacart: parallel promo-code redemption stacks discounts

- **Vuln type / sub-variant:** Limit-overrun (single-use coupon redeemed multiple times), with a second-round bypass on the "one new-user coupon" rule
- **Target type:** E-commerce account promo-code redemption endpoint
- **Recon signal:** A single-use value-granting action (redeem promo code) tied to account state, an obvious spot for a limit-overrun race.
- **Root cause:** Check-then-act without atomicity. The server checks whether the code is already redeemed, then records it, and parallel requests all pass the check before any write lands. The first fix covered only same-code replays and left the cross-code rule (only one new-user coupon allowed) unprotected.
- **Exploitation steps:**
  1. In the account, open Promo Codes and redeem a known public code (example used: a Dallas-themed 20 code).
  2. Intercept the redeem request in a proxy.
  3. Replay it several times asynchronously (exact count not stated, "multiple").
  4. The code is applied more than once and savings stack. An order was shown with a $31.63 discount from the code applied twice.
  5. After the first fix: redeeming two different new-user codes sequentially made the second overwrite the first ($30 total). Redeeming both concurrently applied both ($50 total), showing the exclusivity rule was also racy.
- **Bypasses used:** Switched from same-code repeat to two different new-user codes sent concurrently to defeat the patch.
- **Impact:** Virtually arbitrary discount on orders (stacked coupon value), direct financial loss.
- **Bounty:** Not disclosed, but Instacart paid twice (initial fix and again after the bypass was fixed).
- **Lesson / pattern:** After a race fix, retest the sibling invariants (uniqueness across different codes, not just per code). Redeem-style endpoints need atomic claim-and-mark, not read then write.
- **Source:**
  - Title: Race Condition in Redeeming Coupons
  - Author: cablej
  - URL: https://hackerone.com/reports/157996
  - Date: 2016-08-09
  - Disclosed report: https://hackerone.com/reports/157996
  - Raw file: `sources/race-conditions/10-h1-157996-coupon-redeem-race.md`

Tags: limit-overrun, unlimited-redemption, check-then-act, financial-loss, non-atomic
