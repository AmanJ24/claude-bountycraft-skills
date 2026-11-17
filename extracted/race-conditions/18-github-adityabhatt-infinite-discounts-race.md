# Practitioner write-up: stacking a coupon via parallel requests (PortSwigger limit-overrun lab)

- **Vuln type / sub-variant:** Limit overrun on discount code application, single-endpoint race (lab walkthrough, not a disclosed bounty).
- **Target type:** E-commerce cart, `POST /cart/coupon` (PortSwigger Academy apprentice lab).
- **Recon signal:** A coupon that is meant to apply once per cart, so the apply endpoint is a check-then-act step on shared state.
- **Root cause:** No locking or atomic check around coupon application, so concurrent requests all see "not yet applied" and each add the discount. Author frames it as TOCTOU and missing atomicity. Caveat: the author's real-world examples (a Tesla free-upgrade purchase in 2020, an Uber promo credit abuse in 2016) are uncited claims in the source and should be treated as unverified.
- **Exploitation steps:**
  1. Log in with the lab account, add the target item to the cart, apply the coupon once with the proxy on.
  2. Capture the coupon POST and send it to Repeater.
  3. Duplicate the tab (the text says both 20 and 40, inconsistent) and put all tabs in one group.
  4. Send the group in parallel.
  5. Multiple success messages appear and the cart total drops by the discount times the number of successes.
  6. Buy the item with store credit. If the burst fails, remove the coupon and re-fire.
- **Bypasses used:** None. Repeater's parallel group send only.
- **Impact:** Lab-level: purchase at far below price. Generalised by the author to financial loss in real shops.
- **Bounty:** n/a (lab write-up).
- **Lesson / pattern:** Beginner-level template for limit overrun: baseline once, duplicate to a tab group, send in parallel, reset the precondition (clear the coupon) between attempts. Suggested fixes: DB locking or atomic operations, idempotency checks, one-time tokens. Low-trust on depth and on its real-world claims.
- **Source:**
  - Title: Bug Bounty Race: Exploiting Race Conditions for Infinite Discounts
  - Author: AdityaBhatt3010
  - URL: https://raw.githubusercontent.com/AdityaBhatt3010/Bug-Bounty-Race-Exploiting-Race-Conditions-for-Infinite-Discounts/main/README.md
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/race-conditions/18-github-adityabhatt-infinite-discounts-race.md`

Tags: limit-overrun, single-endpoint, unlimited-redemption, financial-loss, check-then-act, toctou, non-atomic, methodology, defense
