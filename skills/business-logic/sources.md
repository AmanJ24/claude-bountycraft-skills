# Business Logic skill: sources

Every claim in `SKILL.md` and `references/` should trace back to an entry here. Per-writeup notes are in `../../extracted/business-logic/`. 21 sources (19 full/usable, 2 partial). Five candidates were dropped to the HackerOne sign-in wall / no-longer-public status and not bypassed (see `../../sources/business-logic/_candidates.md`).

| # | Title | Author | URL | Date | Disclosed report | Bounty | Extracted note |
|---|-------|--------|-----|------|------------------|--------|----------------|
| 01 | OLO Total price manipulation using negative quantities | fuzz (Matt Thompson) | https://hackerone.com/reports/364843 | 2018-06-11 | same | awarded, amount hidden | `extracted/business-logic/01-h1-364843-upserve-negative-quantity-price.md` |
| 04 | Parameter tampering: Price Manipulation of Products | ashketchum (Ashish Dhone) | https://hackerone.com/reports/682344 | 2019-08-26 | same | none (duplicate; false positive) | `extracted/business-logic/04-h1-682344-wordpress-price-manipulation.md` |
| 05 | Hunting the 6 most common price manipulation vulnerabilities in e-commerce | blackbird-eu (Intigriti) | https://www.intigriti.com/researchers/blog/hacking-tools/top-6-price-manipulation-vulnerabilities-ecommerce | 2024-02-05 | none | n/a (reference) | `extracted/business-logic/05-intigriti-6-price-manipulation-ecommerce.md` |
| 06 | Price manipulation through shipping parameters | Zyad Ibrahim (@zyad_ibrahim333) | https://medium.com/@zyad_ibrahim333/how-i-discovered-a-business-logic-vulnerability-that-allowed-price-manipulation-through-59d4a8c36cea | 2025-11-24 | none | paid, amount not shown | `extracted/business-logic/06-medium-zyad-shipping-param-price.md` |
| 07 | Business Logic, currency arbitrage: pay less than the price in USD | xctzn | https://hackerone.com/reports/1677155 | 2022-08-22 | same | none (Informative; false positive) | `extracted/business-logic/07-h1-1677155-portswigger-currency-arbitrage.md` |
| 08 | Exploiting business logic error vulnerabilities | Ayoub (Intigriti) | https://www.intigriti.com/researchers/blog/hacking-tools/exploiting-business-logic-error-vulnerabilities | 2025-12-09 | none | n/a (reference) | `extracted/business-logic/08-intigriti-exploiting-business-logic-currency.md` |
| 09 | Ethereum account balance manipulation | vicompany | https://hackerone.com/reports/300748 | 2017-12-27 (disc. 2018-03-21) | same | awarded, amount hidden (partial source) | `extracted/business-logic/09-h1-300748-coinbase-ethereum-balance.md` |
| 10 | Amount Manipulation: buy unlimited credits for $1.00 | anonymousmian | https://hackerone.com/reports/277377 | 2017-10-15 | same | none (duplicate; partial source) | `extracted/business-logic/10-h1-277377-inflection-unlimited-credits.md` |
| 11 | Fee discounts can be redeemed many times (unlimited fee-free transactions) | ian | https://hackerone.com/reports/1849626 | 2023-01-28 | same | $5,000 (+2x $50 retest) | `extracted/business-logic/11-h1-1849626-stripe-unlimited-discount.md` |
| 12 | Business Logic Security Cheat Sheet | OWASP Cheat Sheet Series | https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html | n/a | none | n/a (reference) | `extracted/business-logic/12-owasp-business-logic-cheat-sheet.md` |
| 13 | Manipulating response leads to free access to Streamlabs Prime | sudi | https://hackerone.com/reports/1070510 | 2021-01-03 | same | awarded, amount hidden; Medium | `extracted/business-logic/13-h1-1070510-logitech-streamlabs-prime-response.md` |
| 14 | Business Logic Bypass: "Read Access" role without Pro plan | ziadmomen | https://hackerone.com/reports/3591764 | 2026-03-08 | same | none (VDP); Medium | `extracted/business-logic/14-h1-3591764-lovable-access-level-bypass.md` |
| 15 | Add more seats by paying less via PUT /v2/seats | life__001 | https://hackerone.com/reports/1446090 | 2022-01-11 | same | awarded, amount hidden; Medium | `extracted/business-logic/15-h1-1446090-krisp-seats-paying-less.md` |
| 16 | Non-premium user can change retailers to get cashback (Curve) | praseudo7 | https://hackerone.com/reports/672487 | 2019-08-13 | same | awarded + bonus, amounts hidden; Medium | `extracted/business-logic/16-h1-672487-curve-nonpremium-cashback.md` |
| 18 | Incorrect logic when buying one more license extends existing expiry | liru | https://hackerone.com/reports/2461737 | 2024-04-13 | same | none (Informative; no net benefit) | `extracted/business-logic/18-h1-2461737-portswigger-license-expiry.md` |
| 19 | Business Logic Error: bypass of OTP verification during signup (hover.com) | c0rvuz | https://hackerone.com/reports/3255473 | 2025-07-16 | same | none (VDP) | `extracted/business-logic/19-h1-3255473-tucows-otp-signup-bypass.md` |
| 20 | Claiming the listing of a non-delivery restaurant through OTP manipulation | ashoka_rao | https://hackerone.com/reports/1330529 | 2021-09-05 | same | $3,250 total; Critical 9.1 | `extracted/business-logic/20-h1-1330529-eternal-otp-claim-listing.md` |
| 21 | Business Logic error leads to bypass 2FA requirement | abdulprkr | https://hackerone.com/reports/2571981 | 2024-06-24 | same | none (duplicate; intended scope) | `extracted/business-logic/21-h1-2571981-hackerone-2fa-requirement-bypass.md` |
| 23 | Examples of business logic vulnerabilities | PortSwigger Web Security Academy | https://portswigger.net/web-security/logic-flaws/examples | n/a | none | n/a (reference + lab taxonomy) | `extracted/business-logic/23-portswigger-logic-flaws-examples.md` |
| 25 | $1,500 Bounty: Business Logic Flaw in Project Secrets Management | Abhi Sharma (@a13h1) | https://infosecwriteups.com/1-500-bounty-business-logic-flaw-in-project-secrets-management-4b985175071e | 2025-09-19 | none | $1,500 | `extracted/business-logic/25-infosecwriteups-secrets-mgmt-ui-api-mismatch.md` |
| 26 | Top 10 Overlooked Business Logic Flaws in Web Applications | Spectat0rguy (bitpanic) | https://bitpanic.medium.com/top-10-overlooked-business-logic-flaws-in-web-applications-6745d42f3786 | 2025-01-19 | none | n/a (reference) | `extracted/business-logic/26-medium-bitpanic-top10-overlooked-bl.md` |

Dropped (HackerOne sign-in wall / not publicly disclosed, not bypassed): Shipt #388564, Adobe #218748 (both redundant with price coverage), New Relic #200576 / #1089467 / #219356.

Notes:
- #04 and #07 are kept deliberately as false-positive / rejected examples (SKILL.md section 5).
- #18 is kept as a "no net benefit" example.
- #09 and #10 are partial (limited public detail); claims drawn from them are held to what the public summary supports.
