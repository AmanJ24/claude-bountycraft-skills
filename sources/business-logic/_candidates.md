# Business Logic: candidate writeups (Find + Filter steps)

Find/Filter done 2026-11-17. Filter criteria: real root cause + impact (not "found a weird param"),
ideally links a disclosed report. Covet sub-variant diversity across the business-logic taxonomy:
price manipulation (negative qty, fraction/rounding, parameter tampering, integer overflow, currency
confusion), workflow / multi-step bypass, flawed state machine, premium/subscription/license bypass,
OTP/2FA/auth-flow logic, discount/coupon abuse, balance/credit manipulation, privilege/role logic,
response-manipulation client-trust, infinite-money chains.
Target: 15 to 25 kept. Ask before scraping more than ~25 pages in one run. (24 selected.)

Status: candidate | keep | drop (reason) | scraped

| #  | Title | Author | URL | Platform | Report link | Sub-variant | Status |
|----|-------|--------|-----|----------|-------------|-------------|--------|
| 1  | OLO Total price manipulation using negative quantities | maxwell | https://hackerone.com/reports/364843 | hackerone | same | price: negative quantity | candidate |
| 2  | Price manipulation via fraction values (parameter tampering) | kodaihphmc | https://hackerone.com/reports/388564 | hackerone | same | price: fraction/rounding | candidate |
| 3  | Parameter tampering can result in product price manipulation | ddworken | https://hackerone.com/reports/218748 | hackerone | same | price: parameter tampering | candidate |
| 4  | Parameter tampering: Price Manipulation of Products | — | https://hackerone.com/reports/682344 | hackerone | same | price: parameter tampering | candidate |
| 5  | Hunting the 6 most common price manipulation vulnerabilities in e-commerce | Intigriti | https://www.intigriti.com/researchers/blog/hacking-tools/top-6-price-manipulation-vulnerabilities-ecommerce | intigriti | none | price: methodology (ref) | candidate |
| 6  | Business Logic Vulnerability: price manipulation through shipping parameters | Zyad Ibrahim | https://medium.com/@zyad_ibrahim333/how-i-discovered-a-business-logic-vulnerability-that-allowed-price-manipulation-through-59d4a8c36cea | medium | none | price: shipping param | candidate |
| 7  | Business Logic, currency arbitrage - pay less than the price in USD | albinowax/others | https://hackerone.com/reports/1677155 | hackerone | same | currency confusion | candidate |
| 8  | Exploiting business logic error vulnerabilities (currency confusion) | Intigriti (Ayoub) | https://www.intigriti.com/researchers/blog/hacking-tools/exploiting-business-logic-error-vulnerabilities | intigriti | none | currency (ref) | candidate |
| 9  | Ethereum account balance manipulation | savainguyen | https://hackerone.com/reports/300748 | hackerone | same | balance manipulation | candidate |
| 10 | Amount Manipulation: buy unlimited credits for $1.00 | — | https://hackerone.com/reports/277377 | hackerone | same | price: amount tamper | candidate |
| 11 | [Stripe] fee discount can be redeemed unlimited times | ian | https://hackerone.com/reports/1849626 | hackerone | same | discount: unlimited redemption | candidate |
| 12 | Business Logic Security Cheat Sheet (abuse patterns, coupon stacking) | OWASP | https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html | owasp | none | taxonomy/defense (ref) | candidate |
| 13 | Manipulating response leads to free access to Streamlabs Prime | — | https://hackerone.com/reports/1070510 | hackerone | same | premium bypass: response manip | candidate |
| 14 | Business Logic Bypass: free-tier users bypass Pro via access_level param | — | https://hackerone.com/reports/3591764 | hackerone | same | subscription bypass: param | candidate |
| 15 | Add more seats by paying less via PUT /v2/seats request manipulation | — | https://hackerone.com/reports/1446090 | hackerone | same | billing: seats/quantity | candidate |
| 16 | Non-premium user can change retailers to get cashback (Curve) | praseudo7 | https://hackerone.com/reports/672487 | hackerone | same | premium bypass: missing check | candidate |
| 17 | Logic flaw enables restricted account to access account license key | — | https://hackerone.com/reports/200576 | hackerone | same | privilege: restricted-account read | candidate |
| 18 | Incorrect logic when buying one more license extends existing expiry | — | https://hackerone.com/reports/2461737 | hackerone | same | license: date logic | candidate |
| 19 | Business Logic Error: bypass of OTP verification during signup (hover.com) | — | https://hackerone.com/reports/3255473 | hackerone | same | OTP bypass: signup | candidate |
| 20 | Claiming the listing of a non-delivery restaurant through OTP manipulation | — | https://hackerone.com/reports/1330529 | hackerone | same | OTP manipulation: claim | candidate |
| 21 | Business Logic error leads to bypass 2FA requirement | — | https://hackerone.com/reports/2571981 | hackerone | same | 2FA requirement bypass | candidate |
| 22 | Account Takeover via Email ID Change and Forgot Password | — | https://hackerone.com/reports/1089467 | hackerone | same | workflow: email-change -> ATO | candidate |
| 23 | Examples of business logic vulnerabilities (taxonomy + PortSwigger labs) | PortSwigger | https://portswigger.net/web-security/logic-flaws/examples | portswigger | none | taxonomy + labs (ref) | candidate |
| 24 | Manipulation of submit payment request -> paid services for free/reduced | — | https://hackerone.com/reports/219356 | hackerone | same | workflow: payment step tamper | candidate |

Notes:
- The ethicalrohitt/hackerone-reports TOPBUSINESSLOGIC.md list is noisy (many entries are keyword
  matches on "...function" C-library bugs, not logic flaws). Selected only genuine logic bugs above.
- Authors marked "—" to be filled from the report page during scrape.
- References (#5, #8, #12, #23) are methodology/taxonomy, kept deliberately for the synthesis spine;
  the rest are real findings with concrete root cause + impact.
- Backlog if gaps surface: Frontegg PATCH API-key escalation (#2149124), Nextcloud remove-admins
  (#145745), DoD 2FA response-manipulation (#2962527), Legal Robot change-password logic inversion
  (#255679), Kraden subscription flaw (#1505189).

## Scrape resolution (2026-11-17)
21 sources saved in `sources/business-logic/`. Scraped via 3 Sonnet subagents + 2 self-scraped backfills.
- **Saved full/usable (19):** 01, 04, 05, 06, 07, 08, 11, 12, 13, 14, 15, 16, 18, 19, 20, 21, 23, 25, 26.
- **Saved partial (2):** 09 (Coinbase: only the two H1 summaries + timeline render), 10 (Inflection: vendor summary only, researcher body withheld; H1 weakness label is actually IDOR).
- **Dropped, HackerOne sign-in wall / not publicly disclosed (5):** 02 Shipt #388564, 03 Adobe #218748
  (both redundant with price coverage anyway), 17 New Relic #200576, 22 New Relic #1089467,
  24 New Relic #219356. Confirmed not retrievable (sign-in page on stealth retry; `.json` 404). Not bypassed.
- **Backfills added (2):** 25 infosecwriteups Abhi Sharma $1,500 UI-vs-API authorization mismatch
  (privilege logic, excessive client-side trust); 26 bitpanic "Top 10 Overlooked" taxonomy
  (workflow bypass, coupon abuse, order-of-operations) to shore up the workflow/state-machine spine
  lost with the New Relic trio.
- Bounties captured: 11 Stripe $5,000 (+2x$50 retest); 20 Eternal $3,250; 25 $1,500; others awarded
  with amount hidden (01, 13, 15, 16) or none/informative (04, 07, 14, 18, 21).

## Payment-bypass backfill (2026-11-17)
After the skill was committed (2e32ee9), added 3 publicly-readable payment-bypass sources to deepen
that sub-variant (user asked specifically about payment bypass):
- 27 Jack Cable / Lightning Security: webhook/IPN spoofing (forged {"status":"success"} to the merchant webhook).
- 28 Jsmon: payment-gateway-bypass taxonomy (param tampering, return-URL manipulation, webhook spoofing, currency swap, confirm-payment race).
- 29 Amjad Ali (PayU/government site): replaying a captured real gateway success callback (valid hash) for a new unpaid order.
Also confirmed via the user's logged-in HackerOne session that the 5 previously-walled reports
(02 Shipt, 03 Adobe, 17/22/24 New Relic) are "Access denied" even when authenticated = limited
disclosure, genuinely unreachable, not bypassed.
