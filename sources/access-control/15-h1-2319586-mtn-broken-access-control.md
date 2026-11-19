---
title: "Broken Access Control(Horizontal Privilege Escalation)."
author: aliyueka
url: https://hackerone.com/reports/2319586
date_published: 2024-01-15
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/2319586
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

70

[#2319586](https://hackerone.com/reports/2319586)

Copy report idCopy report id

Broken Access Control(Horizontal Privilege Escalation).

- Share:

Report

Timeline

[![aliyueka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/dd1dfc8fc7352772e4a8994c07e1d628d34edc74db48377c707f10829dc23d53)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

submitted a report to [**MTN Group**](https://hackerone.com/mtn_group).

January 15, 2024, 2:51pm UTC

MenuMenu

SUMMARY
access controls are broken, unauthorized users may gain access to sensitive information, modify data, or perform actions that they shouldn't be allowed to. This can lead to various security risks, including data breaches, unauthorized privilege escalation, and other malicious activities.

STEPS TO REPRODUCE
STEP 1:
Go to [https://mtn.ng/offers/](https://mtn.ng/offers/)

imageimage

**Image**•740.00 KiB•F2982514: STEP1.PNG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Enter your number and click on Submit Button

imageimage

**Image**•740.65 KiB•F2982517: STEP2.PNG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Click on Ok

imageimage

**Image**•497.66 KiB•F2982518: STEP3.PNG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

STEP 2:
Enter the OTP code sent to your number

imageimage

**Image**•744.19 KiB•F2982521: STEP4.PNG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

Click on Validate

STEP 3:
MTN offer dashboard will automatically display

imageimage

**Image**•752.36 KiB•F2982526: STEP5.PNG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

[https://mtn.ng/offers/list?phone=2348160817474](https://mtn.ng/offers/list?phone=2348160817474)

STEP 4:
I changed the number that i logged in with my alternative number and it works successfully

imageimage

**Image**•747.68 KiB•F2982536: STEP6.PNG

Open image in fullOpen image in full

[Download mediaDownload media]([REDACTED-SIGNED-URL])

Copy reference IDCopy reference ID

CollapseCollapse

![]([REDACTED-SIGNED-URL])

[https://mtn.ng/offers/list?phone=2349138557692](https://mtn.ng/offers/list?phone=2349138557692)

In this situation an attacker change the phone number to number of his choice

Example:
If you click on this link you will have access to my MTN number without an authentication
[https://mtn.ng/offers/list?phone=2349138557692](https://mtn.ng/offers/list?phone=2349138557692)

## Impact

This vulnerability allow an attacker to access any MTN number in Nigeria and allow threat actors to subscribe data or airtime to the victims.

It can also allow attackers to send messages of their choice to their targeted victims and the victims might think that the message come from MTN.

attachmentsattachments

**7 attachments**

CollapseCollapse

- F2982514: [STEP1.PNG]([REDACTED-SIGNED-URL] "STEP1.PNG")

- F2982517: [STEP2.PNG]([REDACTED-SIGNED-URL] "STEP2.PNG")

- F2982518: [STEP3.PNG]([REDACTED-SIGNED-URL] "STEP3.PNG")

- F2982521: [STEP4.PNG]([REDACTED-SIGNED-URL] "STEP4.PNG")

- F2982526: [STEP5.PNG]([REDACTED-SIGNED-URL] "STEP5.PNG")

- F2982536: [STEP6.PNG]([REDACTED-SIGNED-URL] "STEP6.PNG")

- F2982563: [MTN\_NG.mp4]([REDACTED-SIGNED-URL] "MTN_NG.mp4")


[![Vanessa](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/vsekay)

[vsekay](https://hackerone.com/vsekay)

changed the status to **Triaged**.

[January 22, 2024, 9:06am UTC](https://hackerone.com/reports/2319586#activity-25639733)

MenuMenu

Incident has been assigned for investigation.

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[January 25, 2024, 4pm UTC](https://hackerone.com/reports/2319586#activity-25706587)

MenuMenu

Ok

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[February 1, 2024, 12:10pm UTC](https://hackerone.com/reports/2319586#activity-25808612)

MenuMenu

Hello Team, is there any updates because i was trying to check if the bug exist or not but it seems bug has been fixed by the team.
Please let me know, if there any updates

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[February 8, 2024, 8:39pm UTC](https://hackerone.com/reports/2319586#activity-25934861)

MenuMenu

Any Update?

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[February 19, 2024, 10:20am UTC](https://hackerone.com/reports/2319586#activity-26097401)

MenuMenu

Any Update please

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[March 9, 2024, 9:29pm UTC](https://hackerone.com/reports/2319586#activity-26430499)

MenuMenu

You're not responding!!!

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[April 12, 2024, 9:44pm UTC](https://hackerone.com/reports/2319586#activity-26981763)

MenuMenu

Good Day, I have not received a respond from you.

[![Sly](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/slydingdong1)

[slydingdong1](https://hackerone.com/slydingdong1)

MTN Group staff

posted a comment.

[April 19, 2024, 12:51pm UTC](https://hackerone.com/reports/2319586#activity-27095122)

MenuMenu

Number change after successful logon has been blocked and hence impossible to make any transaction with any other MTN Number , please test

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[April 20, 2024, 10:32pm UTC](https://hackerone.com/reports/2319586#activity-27110516)

MenuMenu

Yes, it has been patched.

[slydingdong1](https://hackerone.com/slydingdong1)

MTN Group staff

closed the report and changed the status to **Resolved**.

[April 24, 2024, 8:15am UTC](https://hackerone.com/reports/2319586#activity-27165118)

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

posted a comment.

[December 31, 2024, 3:30pm UTC](https://hackerone.com/reports/2319586#activity-31728893)

MenuMenu

Can you disclose this report?

[![Eka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/aliyueka)

[aliyueka](https://hackerone.com/aliyueka)

requested to disclose this report.

[January 1, 2025, 11:09am UTC](https://hackerone.com/reports/2319586#activity-31735150)

MenuMenu

Can you disclose this report?

This report has been disclosed.

[January 31, 2025, 11:09am UTC](https://hackerone.com/reports/2319586#activity-32275875)

Reported on

January 15, 2024, 2:51pm UTC

Reported by

![aliyueka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/dd1dfc8fc7352772e4a8994c07e1d628d34edc74db48377c707f10829dc23d53)

[aliyueka](https://hackerone.com/aliyueka)

Reported to

**[MTN Group](https://hackerone.com/mtn_group)**

Managed

Participants

[![aliyueka](https://profile-photos.hackerone-user-content.com/variants/vgvgksh213r12o2uo64rko75n59i/dd1dfc8fc7352772e4a8994c07e1d628d34edc74db48377c707f10829dc23d53)](https://hackerone.com/aliyueka)

[![slydingdong1](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/slydingdong1)

[![vsekay](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/vsekay)

Report Id

[#2319586](https://hackerone.com/reports/2319586)

Resolved

Severity

Medium (6.5)

* * *

Disclosed

January 31, 2025, 11:09am UTC

Weakness

Improper Access Control - Generic

CVE ID

None

* * *

Bounty

None

Retest

None

Total reward

None

* * *

Account details

None

* * *

StripeM-Inner

## Validation Agent Reasoning

CloseClose

Disagree with the reasoning?Let us know

## Exploit Agent Reasoning

CloseClose

## Linear Agent Reasoning

CloseClose