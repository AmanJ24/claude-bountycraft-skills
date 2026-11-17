---
title: "Add more seats by paying less via PUT /v2/seats request manipulation"
author: life__001
url: https://hackerone.com/reports/1446090
date_published: 2022-01-11
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1446090
bounty: not stated (bounty awarded, amount hidden)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

30

[#1446090](https://hackerone.com/reports/1446090)

Copy report idCopy report id

Add more seats by paying less via PUT /v2/seats request manipulation

- Share:

Report

Timeline

[![life__001](ATTACHMENT_URL_REDACTED)](https://hackerone.com/life__001)

[life\_\_001](https://hackerone.com/life__001)

submitted a report to [**Krisp**](https://hackerone.com/krisp).

January 11, 2022, 2:51am UTC

MenuMenu

## Summary:

I could not fully test this vulnerability because the test plan must be completed for the payment process, that is, 30 days. But the price value in api also changes and if payment is made according to this value, wrong billing will occur.

The annual pro option for Team plan billing is $60 per seat. However, if the user enters a decimal number instead of an integer while adding a seat, the number is rounded up, but the price is only multiplied by the integer part. For example it would be like this :

CodeCode

**Code**•202 Bytes

Unwrap linesUnwrap lines

DownloadDownload

CopyCopy

CollapseCollapse

1seats =52amount =3003bady.seats =1.145seats += Math.ceil(bady.seats)6// 5 += 27// seats : 7 89amount += Math.floor(bady.seats)\*6010// 300 += 1 \* 6011// amount : 360

## Steps To Reproduce:

- Register the app and finish the installation. [help document](https://help.krisp.ai/hc/en-us/articles/360017564739-Creating-a-Krisp-personal-account)
- Create a new team.
- Go to billing and listen to traffic with burp.
- Add seat and capture the request with burp.
- Replace the number of seats with 1.9
- You will see that you have added 2 seats but the price has increased by $60.

We can reduce the price by adding and deleting seats.

Poc video -\|

videovideo

**Video**•51.26 MiB•F1574747: k.mp4

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

## Impact

Attacker can manipulate membership price

attachmentsattachments

**1 attachment**

CollapseCollapse

- F1574747: [k.mp4](ATTACHMENT_URL_REDACTED)


[![](https://profile-photos.hackerone-user-content.com/variants/t29wuhl28nb6sqlcvm8pgg6o9k0j/e68529859b080f77ebfd9e0a1acafdf062bd4c43f1db58e7e55c81e6ceae2d0d)](https://hackerone.com/h1_analyst_indy)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

posted a comment.

[January 11, 2022, 9:10am UTC](https://hackerone.com/reports/1446090#activity-14987395)

MenuMenu

Hi [@life\_\_001](https://hackerone.com/life__001),

Thank you for your submission. I hope you are well. Your report is currently being reviewed and the HackerOne triage team will get back to you once there is additional information to share.

Have a great day!

Kind regards,
[@chapeau](https://hackerone.com/chapeau)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

changed the report title from **Billing price manipulation** to **Add more seats by paying less via PUT /v2/seats request manipulation**.

[January 11, 2022, 10am UTC](https://hackerone.com/reports/1446090#activity-14989924)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

updated the severity to
medium(5.0)
.

[January 11, 2022, 10:02am UTC](https://hackerone.com/reports/1446090#activity-14989945)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

added weakness "Improper Input Validation".

[January 11, 2022, 10:02am UTC](https://hackerone.com/reports/1446090#activity-14989946)

[![](https://profile-photos.hackerone-user-content.com/variants/t29wuhl28nb6sqlcvm8pgg6o9k0j/e68529859b080f77ebfd9e0a1acafdf062bd4c43f1db58e7e55c81e6ceae2d0d)](https://hackerone.com/h1_analyst_indy)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

changed the status to **Triaged**.

[January 11, 2022, 10:04am UTC](https://hackerone.com/reports/1446090#activity-14990020)

MenuMenu

Hello [@life\_\_001](https://hackerone.com/life__001),

Thank you for your submission! I'm able to reproduce the issue in your report and have submitted it to the appropriate remediation team for review. The team will review the review and validate if the manipulated seat number is effective, not just the displayed value. They will let us know the final ruling on this report. Please note that the status and severity are subject to change.

Thanks,
[@chapeau](https://hackerone.com/chapeau)

[![Davit](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/noisekiller)

[noisekiller](https://hackerone.com/noisekiller)

Krisp staff

changed the status to **Needs more info**.

[January 11, 2022, 10:58am UTC](https://hackerone.com/reports/1446090#activity-14991077)

MenuMenu

Hi [@life\_\_001](https://hackerone.com/life__001),

We were able to reproduce the issue,

imageimage

**Image**•142.17 KiB•F1575230: Screen\_Shot\_2022-01-11\_at\_14.49.36.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

but it seems to affect the frontend only, as when you proceed to checkout (adding a credit card), it shows the actual bill

imageimage

**Image**•261.88 KiB•F1575231: Screen\_Shot\_2022-01-11\_at\_14.49.44.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

Have you tried proceeding to checkout ?

Thanks for your interest in our BB program,
Cheers,
Davit.

attachmentsattachments

**2 attachments**

CollapseCollapse

- F1575230: [Screen\_Shot\_2022-01-11\_at\_14.49.36.png](ATTACHMENT_URL_REDACTED)

- F1575231: [Screen\_Shot\_2022-01-11\_at\_14.49.44.png](ATTACHMENT_URL_REDACTED)


[![Mevlüt Akçam](ATTACHMENT_URL_REDACTED)](https://hackerone.com/life__001)

[life\_\_001](https://hackerone.com/life__001)

changed the status to **New**.

[January 11, 2022, 11:57am UTC](https://hackerone.com/reports/1446090#activity-14992606)

MenuMenu

Hi [@noisekiller](https://hackerone.com/noisekiller) ,

Firstly i can't add cards because stripe is not live mod. but even if I add a card it doesn't process any payment. Already tells us that the payment will not be made (`Changes to your credit card are effective immediately. Your next charge will be on 10 Feb, 2022.`) And I don't think it only affects the front end because the price returned from the api is the manipulated value.

imageimage

**Image**•130.04 KiB•F1575325: k.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

I have notified you directly because I could not process the payment .if i could do that i wouldn't bother you .
If you have any questions don't hesitate to ask.

Thanks , [@life\_\_001](https://hackerone.com/life__001)

attachmentsattachments

**1 attachment**

CollapseCollapse

- F1575325: [k.png](ATTACHMENT_URL_REDACTED)


[![Davit](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/noisekiller)

[noisekiller](https://hackerone.com/noisekiller)

Krisp staff

changed the status to **Triaged**.

[January 12, 2022, 9:39am UTC](https://hackerone.com/reports/1446090#activity-15006781)

MenuMenu

Hi [@life\_\_001](https://hackerone.com/life__001),

We were able to proceed with checkout and verified the issue,
p.s you can also test checkout process on our staging environment (stage.account.krisp.ai) with stripe test cards ( [https://stripe.com/docs/testing#cards](https://stripe.com/docs/testing#cards)),

Thanks for your interest in our BB program,
Have a nice day ahead !

Cheers,
Davit.

[Krisp](https://hackerone.com/krisp)

rewarded [life\_\_001](https://hackerone.com/life__001) with a bounty.

[January 12, 2022, 9:40am UTC](https://hackerone.com/reports/1446090#activity-15006783)

[![Mevlüt Akçam](ATTACHMENT_URL_REDACTED)](https://hackerone.com/life__001)

[life\_\_001](https://hackerone.com/life__001)

posted a comment.

[January 12, 2022, 10:58am UTC](https://hackerone.com/reports/1446090#activity-15008312)

MenuMenu

Hi [@chapeau](https://hackerone.com/chapeau) ,
thank you for helping me. I think this vulnerability is high level because users can make the price $0. I did not specify the level because I could not verify the vulnerability. can you level up [@chapeau](https://hackerone.com/chapeau) ?

[![](https://profile-photos.hackerone-user-content.com/variants/t29wuhl28nb6sqlcvm8pgg6o9k0j/e68529859b080f77ebfd9e0a1acafdf062bd4c43f1db58e7e55c81e6ceae2d0d)](https://hackerone.com/h1_analyst_indy)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

posted a comment.

[January 12, 2022, 11:19am UTC](https://hackerone.com/reports/1446090#activity-15008740)

MenuMenu

Hi [@life\_\_001](https://hackerone.com/life__001),

Thanks for your comment! The severity of the vulnerability is calculated by CVSS 3.0 Calculator. I'd like to share my thoughts on some metrics:

- Privileges Required: Low. Because the manipulated request needs a Bearer token, hence an authenticated user is required to send the request.
- Confidentiality & Availability: Both none. No sensitive information is disclosed by this vulnerability and has no impact on availability.
- Integrity: Low. Data modification is partially possible. Only the seat number can be manipulated and causing an impact on the price calculation.

Let me know if you have further questions regarding CVSS calculation.

Kind regards,
[@chapeau](https://hackerone.com/chapeau)

[![Mevlüt Akçam](ATTACHMENT_URL_REDACTED)](https://hackerone.com/life__001)

[life\_\_001](https://hackerone.com/life__001)

posted a comment.

[January 12, 2022, 11:43am UTC](https://hackerone.com/reports/1446090#activity-15009169)

MenuMenu

Hi [@chapeau](https://hackerone.com/chapeau) ,

Thanks for the quick response. I think integrity should be at a high. Because data affected here has a financial impact

[![](https://profile-photos.hackerone-user-content.com/variants/t29wuhl28nb6sqlcvm8pgg6o9k0j/e68529859b080f77ebfd9e0a1acafdf062bd4c43f1db58e7e55c81e6ceae2d0d)](https://hackerone.com/h1_analyst_indy)

[h1\_analyst\_indy](https://hackerone.com/h1_analyst_indy)

HackerOne triage

posted a comment.

[January 12, 2022, 4:17pm UTC](https://hackerone.com/reports/1446090#activity-15013173)

MenuMenu

Hi [@life\_\_001](https://hackerone.com/life__001),

According to the CVSS metric description, the High integrity impact means:

> There is a total loss of integrity, or a complete loss of protection. For example, the attacker is able to modify any/all files protected by the impacted component. Alternatively, only some files can be modified, but malicious modification would present a direct, serious consequence to the impacted component.

I think it's clear that this vulnerability can't cause a total loss of integrity. The point that can be discussed here is "...a direct, serious consequence to the impacted component". The vulnerability can result in some fraud cases, meaning the final impact is earning less money. In my opinion, this doesn't consider a critical consequence.

Nevertheless, after the team reviewed this report, they will assess the integrity impact accurately.

Kind regards,
[@chapeau](https://hackerone.com/chapeau)

[noisekiller](https://hackerone.com/noisekiller)

Krisp staff

closed the report and changed the status to **Resolved**.

[April 4, 2022, 9:16am UTC](https://hackerone.com/reports/1446090#activity-16142635)

[life\_\_001](https://hackerone.com/life__001)

requested to disclose this report.

[May 22, 2022, 1:33pm UTC](https://hackerone.com/reports/1446090#activity-16811450)

[![Davit](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/noisekiller)

[noisekiller](https://hackerone.com/noisekiller)

Krisp staff

posted a comment.

[June 20, 2022, 11:33am UTC](https://hackerone.com/reports/1446090#activity-17208539)

MenuMenu

Hi [@life\_\_001](https://hackerone.com/life__001),

do you want a full disclosure or a partial one ?

Best,
[@noisekiller](https://hackerone.com/noisekiller)

[![Mevlüt Akçam](ATTACHMENT_URL_REDACTED)](https://hackerone.com/life__001)

[life\_\_001](https://hackerone.com/life__001)

posted a comment.

[June 20, 2022, 1:09pm UTC](https://hackerone.com/reports/1446090#activity-17210332)

MenuMenu

Hi [@noisekiller](https://hackerone.com/noisekiller) ,

I want full disclosure ,thanks

[noisekiller](https://hackerone.com/noisekiller)

Krisp staff

agreed to disclose this report.

[June 20, 2022, 3:41pm UTC](https://hackerone.com/reports/1446090#activity-17212937)

This report has been disclosed.

[June 20, 2022, 3:41pm UTC](https://hackerone.com/reports/1446090#activity-17212938)

Reported on

January 11, 2022, 2:51am UTC

Reported by

![life__001](ATTACHMENT_URL_REDACTED)

[life\_\_001](https://hackerone.com/life__001)

Reported to

**[Krisp](https://hackerone.com/krisp)**

Participants

[![life__001](ATTACHMENT_URL_REDACTED)](https://hackerone.com/life__001)

[![noisekiller](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/noisekiller)

[![h1_analyst_indy](https://profile-photos.hackerone-user-content.com/variants/t29wuhl28nb6sqlcvm8pgg6o9k0j/33193aed4df168c753cb4b15e6d5d8d6ba053ba5ee3e8d1478c137162601abe9)](https://hackerone.com/h1_analyst_indy)

Report Id

[#1446090](https://hackerone.com/reports/1446090)

Resolved

Severity

Medium (5.0)

* * *

Disclosed

June 20, 2022, 3:41pm UTC

Weakness

Improper Input Validation

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

## Validation Agent Reasoning

CloseClose

Disagree with the reasoning?Let us know

## Exploit Agent Reasoning

CloseClose

## Linear Agent Reasoning

CloseClose

StripeM-Inner