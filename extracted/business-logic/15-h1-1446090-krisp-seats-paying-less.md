# Decimal seat count rounds up for seats but floors for price (Krisp)

- **Vuln type / sub-variant:** Inconsistent rounding between quantity and price calculation (fractional quantity input)
- **Target type:** Team plan billing API for adding seats
- **Recon signal:** The seat-adding request carried a numeric seats value from the client. Trying a non-integer was a natural input-validation probe.
- **Root cause:** The server accepted a decimal seat quantity. It used a ceiling when increasing the seat count but a floor when multiplying by the per-seat price, so the two disagreed. The researcher's reconstructed logic showed 1.1 adding 2 seats while charging for 1.
- **Exploitation steps:**
  1. Create a team and open billing, with traffic proxied.
  2. Capture the add-seat request (PUT to the seats endpoint).
  3. Replace the integer seat count with a decimal such as 1.9.
  4. Observe that two seats were added while the price rose by only one seat's cost.
  5. Repeat with add and delete cycles to lower the price further, up to near zero.
- **Bypasses used:** None. Fractional value alone. Initially the vendor suspected a front-end-only display issue, then confirmed the effect carried through checkout on staging with Stripe test cards.
- **Impact:** Underbilling for extra seats, potentially paying $0 for many. Triage rated Medium (5.0) and the reporter argued for higher integrity impact, which triage declined. The researcher could not fully verify actual charges on production, and the vendor verified via checkout.
- **Bounty:** Awarded, amount hidden.
- **Lesson / pattern:** Send decimals, negatives, and huge values to any quantity field. Look for mismatched rounding functions between "units granted" and "units billed". Verify the effect end-to-end through checkout or a test environment before claiming it, since a displayed price alone may be dismissed.
- **Source:**
  - Title: Add more seats by paying less via PUT /v2/seats request manipulation
  - Author: life__001
  - URL: https://hackerone.com/reports/1446090
  - Date: 2022-01-11
  - Disclosed report: https://hackerone.com/reports/1446090
  - Raw file: `sources/business-logic/15-h1-1446090-krisp-seats-paying-less.md`

Tags: fraction-rounding, price-manipulation, parameter-tampering, missing-server-validation, subscription-bypass, financial-loss
