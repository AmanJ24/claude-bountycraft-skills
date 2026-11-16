# Customer-ID IDOR on credit-card API, scaled via IDs decoded from archived reset links

- **Vuln type / sub-variant:** Cross-user object ID substitution (customer identifier in a cookie/API call) with leaked IDs; read-only PII and payment data exposure. Scale achieved through archive recon.
- **Target type:** Account management app with stored payment methods; backend endpoint under a `/credit_cards/v1/` API taking an `external_customer_id`. Private program.
- **Recon signal:** A cookie named `cam` held an ID that matched the `external_customer_id` used in backend API calls, i.e. the client supplies the customer reference. Hunter created attacker and victim accounts to test, adding fake card data to the victim.
- **Root cause:** The cards API trusted the supplied customer ID without verifying that it belonged to the authenticated user.
- **Exploitation steps:**
  1. Capture the attacker's request to the credit-card endpoint and note the `external_customer_id`.
  2. Replace it with the victim's ID; response returned card number (via card token), expiry date and default-card flag.
  3. To prove broader impact, collect archived URLs with `waybackurls` and `gau`, filtering for indexed forgot/reset-password links.
  4. Decode the JWTs in those links; the payloads contained `externalCustomerId` values for other users (tokens expired, so useless for login, but the IDs were valid).
  5. Replay the extracted IDs against the cards endpoint to retrieve other customers' data.
- **Bypasses used:** ID discovery through public web archives and JWT payload decoding rather than guessing.
- **Impact:** Exposure of live payment card data for hundreds of customers (per title); patched within hours.
- **Bounty:** Not stated in source.
- **Lesson / pattern:** Expired tokens still leak identifiers; mine archived reset, referral and tracking URLs for IDs to feed an IDOR.
- **Source:**
  - Title: How I Chained Recon and IDOR to Access 100's of Credit Cards
  - Author: Bhagavan Bollina (XCriminal)
  - URL: https://medium.com/@XCriminal/how-i-chained-recon-and-idor-to-access-100s-of-credit-cards-0ca50eb82a74
  - Date: 2025-06-15
  - Disclosed report: none
  - Raw file: `sources/idor/16-recon-idor-credit-cards.md`

Tags: uuid, wayback-recon, pii-disclosure, enumeration, api
