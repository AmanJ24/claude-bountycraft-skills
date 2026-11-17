---
title: "Race Condition in Redeeming Coupons"
author: cablej
url: https://hackerone.com/reports/157996
date_published: 2016-08-09
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/157996
bounty: not stated (bounty awarded twice by Instacart, amount hidden on report)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Scrape status: FULL (complete report body and timeline rendered; bounty amount hidden by HackerOne). Presigned attachment URLs with tokens replaced by [REDACTED].

Report #157996: Race Condition in Redeeming Coupons
Reported to: Instacart (Managed). Reported by: cablej. Reported on: August 9, 2016, 10:12pm UTC. Disclosed: September 12, 2016, 8:40am UTC.
Status: Resolved. Severity: No rating. Weakness: Violation of Secure Design Principles. CVE: None. Bounty / Total reward: Hidden.

## Report body (cablej, August 9, 2016, 10:12pm UTC)

Hello,

There exists a race condition in redeeming coupons, allowing a user to redeem the same coupon multiple times, and stacking savings added. This allows for a user to get virtually any discount.

POC:

1. Visit your account and select 'Promo Codes'.
2. Select redeem promo code, and add any promo code. For example, I found the code 'dallas20'.
3. Intercept the request using a proxy, and make the request multiple times, asynchronously.
4. The code will be redeemed multiple times.

For an example, see the screenshot attached.

Attachment F110911: screenshot.png [REDACTED presigned URL]

## Timeline

**cablej** (August 9, 2016, 10:16pm UTC): As another example, here's an order I could place, with a discount of $31.63, the coupon code applied twice.
Attachment F110912: screenshot2.png [REDACTED presigned URL]

**mikeinstacart** (August 12, 2016, 6:35pm UTC): Thanks @cablej! We've released a fix for this, can you confirm if the issue is still present?

**cablej** (August 12, 2016, 7:55pm UTC): Great, and thanks for the response. I created a new account and can confirm that a promo code can now only be redeemed once.

**mikeinstacart** closed the report and changed the status to **Resolved**. (August 12, 2016, 7:59pm UTC)

**Instacart** rewarded cablej with a bounty. (August 12, 2016, 7:59pm UTC)

**cablej** requested to disclose this report. (August 13, 2016, 8:40am UTC)

**cablej** (August 24, 2016, 4:15pm UTC): Hey, I was able to find a workaround this fix:

First, I created an account and redeemed the coupon `dallas20`, followed by `SLFriends30`, which caused the second promo code to overwrite the first, giving $30 in coupons.

With another account, I redeemed `SLFriends30` first, but the site did not allow me to add `dallas20`, giving $30 in coupons.

Redeeming both of these asynchronously, however, gave $50 in coupons, applying both. It seems like there are now measures to prevent a coupon from being redeemed twice, but not multiple 'New User' coupons.

**cablej** (September 5, 2016, 11:45am UTC): It seems that this issue is not completely fixed, could you look into this further?

**anf** (September 6, 2016, 12:12am UTC): Hello @cablej, thank you for letting us know, we will look into this.

**cablej** (September 7, 2016, 11:29pm UTC): This report is set to disclose in 5 days. As it may not be completely resolved yet, is there a way to delay the disclosure? If not, I can contact HackerOne.

**anf** (September 8, 2016, 12:19am UTC): No need, we will have it fixed before then. Thanks for checking in with us :)

**instacart-muffy** (September 9, 2016, 5:30pm UTC): This is resolved now - @cablej please reconfirm? Thanks again!

**cablej** (September 9, 2016, 9:59pm UTC): Yep, I can verify that this is fixed. Thanks!

This report has been disclosed. (September 12, 2016, 8:40am UTC)

**Instacart** rewarded cablej with a bounty. (September 13, 2016, 5:28pm UTC)
Hello @cablej, After further review with the team we decided to add another bounty to this report. Thank you for the very thorough reporting!

Participants: cablej, anf, instacart-muffy, mikeinstacart.
