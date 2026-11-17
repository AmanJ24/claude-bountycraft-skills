---
title: "Business Logic, currency arbitrage - Possibility to pay less than the price in USD"
author: xctzn
url: https://hackerone.com/reports/1677155
date_published: 2022-08-22
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1677155
bounty: not stated (None; closed Informative)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #1677155. Reported to PortSwigger Web Security. Status: Informative. Severity: Medium (6.7). Weakness: Business Logic Errors. Bounty: None. Disclosed Oct 26, 2022.

## Timeline

**xctzn** submitted a report to PortSwigger Web Security. August 22, 2022, 10:05pm UTC

Currency fluctuate all the time. Theses days EUR / USD key pair is around 1for1. It was even 1:0.99 when I was writing this report.
Portswigger doesn't change dynamically the price and exchange rate dynamically.

Vulnerability at the following link: https://portswigger.net/buy/pro

When you want to buy a product choose the currency, you can noticed they are fixed and with today difference it's quite a big difference.

## Impact

USD price is 399$USD, while EUR price is 349$.
Therefore someone could just change the price to Euro and pay 347 $USD (349 Euro) instead of 399$(with current rate).

PS: It scale with the price, it could lead to thousands of dollars lost for your company.

---

**albinowax** (PortSwigger staff) closed the report and changed the status to **Informative**. August 24, 2022, 9:25am UTC

Thank you for your report. This is a business decision and not something we regard as a vulnerability.

**xctzn** posted a comment. October 25, 2022, 9:44pm UTC

Good afternoon,

To my surprise it seems that you fixed the business decision following my recommendation.
Why did you patched if it was not a vulnerability (business logic error, as I stated)?

Could you explain it to me ? I really don't get it why you would not accept it because it was clearly a way to pay less and then cut PortSwigger's margin.

If you remain with your actual judgement, could you disclose the report? I'd like to share it with other fellow bug hunter to have their opinion on that.

**albinowax** posted a comment. October 26, 2022, 6:56am UTC

Hi @xctzn, the base price of the product was was changed so naturally we changed the price in all currencies. It's still set statically.

As requested, I'll disclose the report.

albinowax requested to disclose and disclosed this report. October 26, 2022.
