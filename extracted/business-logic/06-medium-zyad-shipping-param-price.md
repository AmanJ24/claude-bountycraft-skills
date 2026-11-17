# Shipping cost and tax cut by swapping the shipping country in the place-order request

- **Vuln type / sub-variant:** Price manipulation via shipping-fee/tax inputs (destination country used as a client-controlled pricing input)
- **Target type:** E-commerce checkout API (place-order endpoint, PayPal payment)
- **Recon signal:** No product price or amount field was editable or reflected, so the hunter shifted to the other terms in the total: shipping and tax. The order request had a shipping country parameter that drives fee calculation.
- **Root cause:** The shipping fee and tax were derived from a client-supplied country value at payment time, without recomputing against the real delivery address. The backend did not tie the fee to the actual destination.
- **Exploitation steps:**
  1. Intercept the place-order request on the checkout API.
  2. Find the shipping country field and change it to a country known to have cheaper shipping.
  3. Proceed to PayPal and observe the lower shipping charge and no tax.
  4. Confirm the real destination address was unchanged, so goods still ship to the original place.
- **Bypasses used:** Parameter tampering (pricing-relevant field, not a price field); decoupling pricing country from delivery address
- **Impact:** Reduced shipping cost and removed tax on the order. Triaged P3, bounty paid (amount not shown).
- **Bounty:** not stated (paid, amount not shown)
- **Lesson / pattern:** When price fields are locked down, test every input that feeds the total (country, region, shipping method, tax flags) and check whether each is re-derived server-side.
- **Source:**
  - Title: How I Discovered a Business Logic Vulnerability That Allowed Price Manipulation Through Shipping Parameters
  - Author: Zyad Ibrahim (@zyad_ibrahim333)
  - URL: https://medium.com/@zyad_ibrahim333/how-i-discovered-a-business-logic-vulnerability-that-allowed-price-manipulation-through-59d4a8c36cea
  - Date: 2025-11-24
  - Disclosed report: none
  - Raw file: `sources/business-logic/06-medium-zyad-shipping-param-price.md`

Tags: price-manipulation, parameter-tampering, client-trust, missing-server-validation, assumption-flaw, financial-loss
