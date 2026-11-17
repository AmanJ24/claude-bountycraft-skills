# Static per-currency list prices allow paying less by choosing a cheaper currency (rejected as business decision)

- **Vuln type / sub-variant:** Currency arbitrage from fixed, non-synced price tables (pricing-policy gap, not input tampering)
- **Target type:** Product purchase page with selectable currency (software license shop)
- **Recon signal:** Exchange rates move constantly but the shop listed fixed prices per currency. Comparing USD and EUR prices against the live rate showed the euro price was effectively lower.
- **Root cause:** Prices for each currency are set statically and not tied to a live exchange rate, so a gap opens whenever the rate drifts.
- **Exploitation steps:**
  1. Open the buy page and compare the USD list price to the EUR list price.
  2. Convert the EUR price at the current rate and note it is below the USD price.
  3. Pay in the cheaper currency.
- **Bypasses used:** none (uses intended currency selector)
- **Impact:** Savings of roughly tens of dollars per license that scales with price. Program judged it a business decision, not a vulnerability. Closed Informative.
- **Bounty:** none
- **Lesson / pattern:** Using the legitimate currency selector is a pricing decision, not an exploit. A real currency bug needs the amount or currency to be attacker-controlled and mismatched (see currency confusion), not just a stale price table. Expect rejection and do not file these.
- **Source:**
  - Title: Business Logic, currency arbitrage - Possibility to pay less than the price in USD
  - Author: xctzn
  - URL: https://hackerone.com/reports/1677155
  - Date: 2022-08-22
  - Disclosed report: https://hackerone.com/reports/1677155
  - Raw file: `sources/business-logic/07-h1-1677155-portswigger-currency-arbitrage.md`

Tags: currency-confusion, price-manipulation, assumption-flaw, financial-loss
