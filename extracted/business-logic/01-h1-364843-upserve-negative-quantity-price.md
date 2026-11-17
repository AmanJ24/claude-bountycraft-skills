# Online ordering total reduced by adding a negative-quantity line item (and client-supplied item price)

- **Vuln type / sub-variant:** Price manipulation via negative quantity, plus tampering of per-item price and tax fields in the order JSON
- **Target type:** Online food-ordering checkout API (restaurant ordering platform, JSON order submission)
- **Recon signal:** The order is submitted as one JSON blob that carries per-item price, quantity, taxes and totals, so every money field is attacker-editable. The hunter tried adding extra items and odd values to see what the total check accepts.
- **Root cause:** The server only checked that the submitted grand total was internally consistent with the submitted line items, fees and taxes. It never validated line quantities as positive, and never recomputed prices from its own catalog. A negative-quantity line simply subtracted from the sum and the arithmetic still "balanced".
- **Exploitation steps:**
  1. Capture the order submission request and replay it from a script.
  2. Keep a real item (2 units, about $12 each) and append a second item with quantity -1 (about $9).
  3. Recompute the total and tax so the totals line up (the API rejects mismatched totals).
  4. Submit. The order is accepted and only the reduced amount (about $18.70) is charged.
  5. Variation: edit the "price" key of an item to a cent, and zero out taxes, again keeping the grand total consistent. The stored order keeps the tampered price.
- **Bypasses used:** Negative quantity; parameter tampering of item price and tax; keeping totals self-consistent to satisfy the integrity check
- **Impact:** Attacker pays less than the true value of the order. Orders also arrive out of balance for the restaurant (a delivery-fee omission gave a second, lower-impact flaw), forcing manual resolution. Triaged Critical (9-10 self-rated range), resolved.
- **Bounty:** not stated (reward granted, amount hidden)
- **Lesson / pattern:** If the server validates "does the total match the lines" instead of "are the lines valid and priced from my database", negative quantities and edited prices will pass.
- **Source:**
  - Title: OLO Total price manipulation using negative quantities
  - Author: fuzz (Matt Thompson)
  - URL: https://hackerone.com/reports/364843
  - Date: 2018-06-11
  - Disclosed report: https://hackerone.com/reports/364843
  - Raw file: `sources/business-logic/01-h1-364843-upserve-negative-quantity-price.md`

Tags: price-manipulation, negative-quantity, parameter-tampering, missing-server-validation, client-trust, financial-loss, assumption-flaw
