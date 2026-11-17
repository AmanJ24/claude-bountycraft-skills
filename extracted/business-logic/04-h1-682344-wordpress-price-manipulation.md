# Cart price and shipping edited in the redirect to PayPal (rejected: IPN validation catches it)

- **Vuln type / sub-variant:** Parameter tampering of amount and shipping in the payment-gateway handoff request (client-side cart-to-PayPal redirect)
- **Target type:** E-commerce checkout built on WooCommerce with PayPal standard redirect
- **Recon signal:** The "Proceed to PayPal" step sends the amount and shipping values in a browser-visible request to the payment provider, so they can be intercepted and edited before reaching PayPal.
- **Root cause (claimed):** Price fields travel through the client to the payment provider. The program's answer was that this is known WooCommerce behavior and the order is validated afterward, so the claim was only apparent, not real.
- **Exploitation steps:**
  1. Add a product to the cart and fill billing details.
  2. Intercept the request made when clicking proceed to PayPal.
  3. Change the item amount and shipping parameters in the request (to a low value or zero).
  4. Forward it so PayPal shows the reduced amount.
- **Bypasses used:** Parameter tampering at the gateway handoff
- **Impact:** Claimed free or cheaper purchase. Not accepted: the instant payment notification check runs after the order and marks a mismatched payment as invalid, so no real price change. Closed as duplicate of a known issue.
- **Bounty:** none (duplicate)
- **Lesson / pattern:** Editing the price in a redirect to the payment processor is only a real bug if the merchant fails to verify the paid amount server-side afterward. Always check whether the order is actually fulfilled, not just whether the gateway page shows the lower number.
- **Source:**
  - Title: Parameter tampering : Price Manipulation of Products
  - Author: ashketchum (Ashish Dhone)
  - URL: https://hackerone.com/reports/682344
  - Date: 2019-08-26
  - Disclosed report: https://hackerone.com/reports/682344
  - Raw file: `sources/business-logic/04-h1-682344-wordpress-price-manipulation.md`

Tags: price-manipulation, parameter-tampering, client-trust, missing-server-validation, assumption-flaw
