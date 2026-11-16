# Acronis storefront: client-supplied price in checkout request reaches payment gateway

- **Vuln type / sub-variant:** Parameter tampering labelled IDOR by the reporter. It is really a trusted-client-value flaw (price), not an object-ownership check; weak fit for IDOR proper.
- **Target type:** E-commerce checkout on a regional storefront (Czech site), request to server before payment gateway redirect.
- **Recon signal:** Intercepting the buy-now click showed the product price as a plain request parameter, unsigned and not encrypted.
- **Root cause:** The server accepted the price from the client and forwarded it to the payment provider instead of looking the price up server-side from the product id. A value of 0 was rejected, but 1 was accepted, so only a minimal sanity check existed.
- **Exploitation steps:**
  1. Pick a product, fill in order details, enable an intercepting proxy.
  2. Click buy now and edit the price parameter in the request before forwarding.
  3. Follow through to the gateway. The reporter showed the card gateway and a bitcoin payment flow both asked for the tampered amount.
  4. Triage first thought only the UI price changed. The reporter stopped before paying and showed gateway pending-payment emails as proof the amount reached billing.
- **Bypasses used:** Using a small non-zero value (1) to pass a zero-value check.
- **Impact:** Attacker can buy products at an arbitrary low price. Never completed a real payment. Rated medium.
- **Bounty:** Awarded, amount not stated in source.
- **Lesson / pattern:** In checkout flows, any amount, currency or item id sent by the client is a candidate. Prove the downstream effect (gateway amount), not just the UI. Note this overlaps business-logic more than classic IDOR.
- **Source:**
  - Title: IDOR vulnerability (Price manipulation)
  - Author: spookhorror
  - URL: https://hackerone.com/reports/1403176
  - Date: reported 2021-11-17, disclosed 2021-11-30
  - Disclosed report: https://hackerone.com/reports/1403176
  - Raw file: `sources/idor/07-h1-1403176-acronis-price-manipulation-idor.md`

Tags: price-manipulation, mass-assignment, api, business-logic
