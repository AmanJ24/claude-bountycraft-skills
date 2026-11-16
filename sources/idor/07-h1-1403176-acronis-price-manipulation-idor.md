---
title: "IDOR vulnerability (Price manipulation)"
author: spookhorror
url: https://hackerone.com/reports/1403176
date_published: 2021-11-17
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1403176
bounty: not stated (awarded, amount hidden)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR vulnerability (Price manipulation)

Reported to: Acronis | Reporter: spookhorror | Reported: 2021-11-17 | Disclosed: 2021-11-30 | State: Resolved | Severity: medium

Timeline

[spookhorror](https://hackerone.com/spookhorror)

submitted a report to [**Acronis**](https://hackerone.com/acronis).

November 17, 2021, 4:39pm UTC

Target: acronis.cz

Step to Reproduce

1.Go to acronis.cz
2.buy any product in this case i am going to buy this [https://www.acronis.cz/produkt/acronis-cyber-protect-home-office/](https://www.acronis.cz/produkt/acronis-cyber-protect-home-office/) for test
3.fill up details
4.go to burpsuite turn on intercept
5.click on buy now
6.check request in intercept change price of product and click on forward

Check out price\_manipulation\_POC.mkv for Practical Demonstration

## Impact

attacker can reduce the price of the Product.


**2 attachments**

- F1517264: [price\_manipulation\_POC.mkv]

- F1517265: [Price\_manipulation.PNG]

[security-bot](https://hackerone.com/security-bot)

posted a comment.

[November 17, 2021, 4:43pm UTC](https://hackerone.com/reports/1403176#activity-14344582)

Hi [@spookhorror](https://hackerone.com/spookhorror)

Thanks for participating in Acronis bug bounty program!
Our security team usually replies within 1-2 business days.

Meanwhile, I'd recommend you to check out our [policy](https://hackerone.com/acronis).

[spookhorror](https://hackerone.com/spookhorror)

posted a comment.

[November 17, 2021, 4:57pm UTC](https://hackerone.com/reports/1403176#activity-14344716)

ok I will wait

[iradostev](https://hackerone.com/iradostev)

Acronis staff

changed the status to **Needs more info**.

[November 18, 2021, 2:27pm UTC](https://hackerone.com/reports/1403176#activity-14359382)

Hi [@spookhorror](https://hackerone.com/spookhorror)

I believe what you performed only impacts the price displayed in UI, not the actual billing information.

If you disagree, please reply with additional information describing your reasoning. Including a working proof-of-concept can be incredibly helpful in our assessment of these claims. Otherwise, feel free to self-close the report.

[spookhorror](https://hackerone.com/spookhorror)

changed the status to **New**.

[November 19, 2021, 5:21am UTC](https://hackerone.com/reports/1403176#activity-14366219)

I think its vulnerability because it's not reflecting it's changing the actual price of the product

when I enter price 0 in parameter server rejected the request and when I enter 1 price in parameter server think its valid and forward the request.
check out
[0kc\_value\_in\_price.mkv.mkv (F1519130)] [1kc\_in\_price.mkv (F1519131)]

and I have just perform same vulnerability again to test and I cancel the payment and i got mail from go pay.
check out
[mail.PNG (F1519134)] [payment.PNG (F1519149)] [payment\_2.PNG (F1519148)]

( Payment is pending because I haven't paid and sorry I can't buy via bitcoin and visa I don't have credit card and visa so sorry )
so here price in payment gateway actually changed because payment gateway got request from website server to pay certain amount of price after manipulating request
I did the same thing and price changed in bitcoin too
check out [bitcoin.PNG (F1519132)]

1\. Server is not verifying whether request is original or not

2.payment request to the server from client its not encrypted.check out [this\_should\_be\_encrypted.PNG (F1519133)]
so anyone can change details


**8 attachments**

- F1519130: [0kc\_value\_in\_price.mkv.mkv]

- F1519131: [1kc\_in\_price.mkv]

- F1519132: [bitcoin.PNG]

- F1519133: [this\_should\_be\_encrypted.PNG]

- F1519134: [mail.PNG]

- F1519135: [server\_does\_not\_verifying\_request.mkv]

- F1519148: [payment\_2.PNG]

- F1519149: [payment.PNG]

[iradostev](https://hackerone.com/iradostev)

Acronis staff

changed the status to **Triaged**.

[November 19, 2021, 9:47am UTC](https://hackerone.com/reports/1403176#activity-14369168)

You are right, [@spookhorror](https://hackerone.com/spookhorror). Thanks for clarification

[spookhorror](https://hackerone.com/spookhorror)

posted a comment.

[November 19, 2021, 10:06am UTC](https://hackerone.com/reports/1403176#activity-14369422)

okay sir thanks for Triaged.

[spookhorror](https://hackerone.com/spookhorror)

posted a comment.

[November 22, 2021, 7:54am UTC](https://hackerone.com/reports/1403176#activity-14393376)

any updates ?

[spookhorror](https://hackerone.com/spookhorror)

posted a comment.

Updated [November 23, 2021, 9:04am UTC](https://hackerone.com/reports/1403176#activity-14410027)

any updates ? [@iradostev](https://hackerone.com/iradostev)

[Acronis](https://hackerone.com/acronis)

rewarded [spookhorror](https://hackerone.com/spookhorror) with a bounty.

[November 23, 2021, 9:42am UTC](https://hackerone.com/reports/1403176#activity-14411457)

Thanks for participating in Acronis bug bounty program! Bounty is transferred to your account.
Happy bug hunting and we are waiting for new reports from you.

[spookhorror](https://hackerone.com/spookhorror)

posted a comment.

[November 23, 2021, 9:47am UTC](https://hackerone.com/reports/1403176#activity-14411516)

thank you sir

[security-bot](https://hackerone.com/security-bot)

closed the report and changed the status to **Resolved**.

[November 25, 2021, 1:13pm UTC](https://hackerone.com/reports/1403176#activity-14442723)

Thanks for catching this bug!

We have fixed it, please confirm that you are no longer able to reproduce the issue.

[spookhorror](https://hackerone.com/spookhorror)

posted a comment.

[November 25, 2021, 1:20pm UTC](https://hackerone.com/reports/1403176#activity-14442792)

I am not able to reproduce the bug its fixed!


**1 attachment**

- F1525680: [fixed.mkv]

[spookhorror](https://hackerone.com/spookhorror)

requested to disclose this report.

[November 26, 2021, 3:56am UTC](https://hackerone.com/reports/1403176#activity-14446625)

[stushieva](https://hackerone.com/stushieva)

Acronis staff

agreed to disclose this report.

[November 30, 2021, 9:17am UTC](https://hackerone.com/reports/1403176#activity-14482350)

This report has been disclosed.

[November 30, 2021, 9:17am UTC](https://hackerone.com/reports/1403176#activity-14482351)

Reported on

November 17, 2021, 4:39pm UTC

Reported by



[spookhorror](https://hackerone.com/spookhorror)

Reported to

**[Acronis](https://hackerone.com/acronis)**

Participants

Report Id

[#1403176](https://hackerone.com/reports/1403176)

Resolved

Severity

Medium (4 ~ 6.9)

* * *

Disclosed

November 30, 2021, 9:17am UTC

Weakness

Insecure Direct Object Reference (IDOR)

CVE ID

None

* * *

Bounty

[Hidden](https://docs.hackerone.com/en/articles/8395720-payment-preferences#h_733fa743ac)

Retest

[Hidden](https://docs.hackerone.com/en/articles/8395720-payment-preferences#h_733fa743ac)

Total reward

[Hidden](https://docs.hackerone.com/en/articles/8395720-payment-preferences#h_733fa743ac)

* * *

Account details

None

* * *
