# Amount manipulation to buy credits for $1 (Inflection, partial)

- **Vuln type / sub-variant:** Price/amount parameter tampering on a credit purchase (claimed). Closed as duplicate, so no validated detail.
- **Target type:** Credit purchase / payment flow
- **Recon signal:** Not stated. The title implies the purchase amount was a client-controlled value.
- **Root cause:** Not stated in public content (source is partial). The researcher's body is withheld, so only the title hints at server trust in a submitted amount. The program labelled the weakness as IDOR.
- **Exploitation steps:**
  1. Not disclosed. Title only suggests altering the amount in the purchase request to pay $1 for a large credit quantity.
- **Bypasses used:** Not stated.
- **Impact:** Claimed unlimited credits for $1. Severity was originally high, then lowered to none by the vendor. Closed as Duplicate of #276787 and disclosed at the researcher's request with the body withheld.
- **Bounty:** None (duplicate).
- **Lesson / pattern:** Low-value as a technique source. It is a reminder that amount/credit-quantity fields on purchase endpoints are a standard first check, and that duplicates are common for obvious payment tampering.
- **Source:**
  - Title: Amount Manipulation Buy Unlimited Credits in just $1.00 (Inflection)
  - Author: anonymousmian
  - URL: https://hackerone.com/reports/277377
  - Date: 2017-10-15
  - Disclosed report: https://hackerone.com/reports/277377
  - Raw file: `sources/business-logic/10-h1-277377-inflection-unlimited-credits.md`

Tags: price-manipulation, parameter-tampering, free-services, financial-loss
