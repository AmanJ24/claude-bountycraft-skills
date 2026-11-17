---
title: "Business Logic Bypass Allows Setting "Read Access" Role Without Pro Plan Subscription"
author: ziadmomen
url: https://hackerone.com/reports/3591764
date_published: 2026-03-08
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/3591764
bounty: not stated
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

78

[#3591764](https://hackerone.com/reports/3591764)

Copy report idCopy report id

Business Logic Bypass Allows Setting “Read Access” Role Without Pro Plan Subscription

- Share:

Report

Timeline

[![ziadmomen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/7d6e0417f1813b18bcb08485a78f3f450b5240c7e7ad6e0c3ca1ab23e56369d2)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

submitted a report to [**Lovable VDP**](https://hackerone.com/lovable-vdp).

March 8, 2026, 8:08am UTC

MenuMenu

## Summary:

A business logic flaw allows a user with a free plan to generate an invitation link that assigns the Read Access (Read) role, even though this role is intended to be restricted to users with a Pro Plan subscription. By manipulating the invitation creation process, an attacker can create an invite link that grants this restricted role without having the required paid subscription.

## Steps To Reproduce:

01. Create two accounts one for the owner and the second one for the user who will be invited to the project.

02. From the owner account create a project.












    imageimage





    **Image**•55.31 KiB•F5489258: image.png






    Open image in fullOpen image in full


    [Download mediaDownload media](ATTACHMENT_URL_REDACTED)


    Copy reference IDCopy reference ID





    CollapseCollapse














    ![](ATTACHMENT_URL_REDACTED)

03. Choose any template, click use template button, name your project, and click Remix button












    imageimage





    **Image**•289.34 KiB•F5489260: image.png






    Open image in fullOpen image in full


    [Download mediaDownload media](ATTACHMENT_URL_REDACTED)


    Copy reference IDCopy reference ID





    CollapseCollapse














    ![](ATTACHMENT_URL_REDACTED)

04. Go to the created project you will find it at `Projects` section.












    imageimage





    **Image**•171.23 KiB•F5489266: image.png






    Open image in fullOpen image in full


    [Download mediaDownload media](ATTACHMENT_URL_REDACTED)


    Copy reference IDCopy reference ID





    CollapseCollapse














    ![](ATTACHMENT_URL_REDACTED)

05. Click on `share`, on the top right corner, and notice in the invite link that you should be on the pro plane to modify the access to `View access`.












    imageimage





    **Image**•518.73 KiB•F5489273: image.png






    Open image in fullOpen image in full


    [Download mediaDownload media](ATTACHMENT_URL_REDACTED)


    Copy reference IDCopy reference ID





    CollapseCollapse














    ![](ATTACHMENT_URL_REDACTED)

06. From invite link click `Disabled`.

07. Return the access right back to `Edit access` while keeping the intercept on.

08. You should notice the `OPTIONS request` has been issued to the projects endpoint. Forward it and wait for the `POST method`.

09. Modify the `access_level` parameter from `write` to `read` and do intercept to the response.












    imageimage





    **Image**•127.62 KiB•F5489341: image.png






    Open image in fullOpen image in full


    [Download mediaDownload media](ATTACHMENT_URL_REDACTED)


    Copy reference IDCopy reference ID





    CollapseCollapse














    ![](ATTACHMENT_URL_REDACTED)

10. Notice that the `read` permission reflected in the response without any issues. Go Back to your project and refresh the page. Click on share and you will find that right access become `View access` while you are on the free plane.

11. Copy the invite link and paste it in the second account , accept the invitation to the project, click on share -> `people you invited`, and you will find that the second account gained access with the `viewer` permission.












    imageimage





    **Image**•43.53 KiB•F5489359: image.png






    Open image in fullOpen image in full


    [Download mediaDownload media](ATTACHMENT_URL_REDACTED)


    Copy reference IDCopy reference ID





    CollapseCollapse














    ![](ATTACHMENT_URL_REDACTED)


Example of the request from which you can modify the access right

CodeCode

**Code**•461 Bytes

Unwrap linesUnwrap lines

DownloadDownload

CopyCopy

CollapseCollapse

1POST /projects/Project-ID/magic-codes HTTP/2
2Host: api.lovable.dev
3Cookie: \[Your-Cookie\]
4User-Agent: \[\]
5Accept: \*/\*
6Accept-Language: en-US,en;q=0.5
7Accept-Encoding: gzip, deflate, br
8Referer: https://lovable.dev/
9Authorization: Bearer \[Your-Authorization-Header\]
10Content-Type: application/json
11Content-Length: 23
12Origin: https://lovable.dev
13Sec-Fetch-Dest: empty
14Sec-Fetch-Mode: cors
15Sec-Fetch-Site: same-site
16Priority: u=0
17Te: trailers
1819{"access\_level":"read"}

## Impact

Impact:

This vulnerability allows users to bypass the platform’s subscription restrictions and access premium features without paying. As a result, attackers can assign roles that should only be available to Pro Plan users, leading to unauthorized use of paid functionality and potential financial impact on the platform due to subscription bypass.
From a security perspective, this mainly affects Integrity and Business Logic enforcement, as the system fails to properly validate plan-based permission restrictions.

attachmentsattachments

**7 attachments**

CollapseCollapse

- F5489258: [image.png](ATTACHMENT_URL_REDACTED)

- F5489260: [image.png](ATTACHMENT_URL_REDACTED)

- F5489266: [image.png](ATTACHMENT_URL_REDACTED)

- F5489273: [image.png](ATTACHMENT_URL_REDACTED)

- F5489311: [image.png](ATTACHMENT_URL_REDACTED)

- F5489341: [image.png](ATTACHMENT_URL_REDACTED)

- F5489359: [image.png](ATTACHMENT_URL_REDACTED)


[![ziad momen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/cbcbf98f1b109b2be47f5597756b8bb14b56c3ba66c64b2d2db1e108bfcc0a32)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

posted a comment.

Deleted [March 8, 2026, 9:31am UTC](https://hackerone.com/reports/3591764#activity-39906602)

_This comment was deleted by the author._

[![](https://profile-photos.hackerone-user-content.com/variants/k5s1d31x68des504elg1u815wli5/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/h1_analyst_diva)

[h1\_analyst\_diva](https://hackerone.com/h1_analyst_diva)

HackerOne triage

posted a comment.

[March 9, 2026, 4:58pm UTC](https://hackerone.com/reports/3591764#activity-39933707)

MenuMenu

Hello [@ziadmomen](https://hackerone.com/ziadmomen),

Thank you for your submission!

Your report has passed the preliminary analyst review. Please note that this does not confirm validation — the status may change after further review.

Next in workflow is for our team to validate and reproduce the issue, evaluating its accuracy and security impact. You will be notified when the team has reviewed and made an assessment on your report.

We’ll keep you updated as the process moves forward. Have a great day!

Thanks,
[@h1\_analyst\_diva](https://hackerone.com/h1_analyst_diva)

[![](https://profile-photos.hackerone-user-content.com/variants/dh7eyfnor7buii9e32tcuvi4crfp/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/h1_analyst_mars)

[h1\_analyst\_mars](https://hackerone.com/h1_analyst_mars)

HackerOne triage

closed the report and changed the status to **Duplicate** ( [#3557062](https://hackerone.com/reports/3557062)).

[March 11, 2026, 1:49pm UTC](https://hackerone.com/reports/3591764#activity-39978488)

MenuMenu

Hi [@ziadmomen](https://hackerone.com/ziadmomen),

Thank you for your submission. After reviewing your report, this issue has been previously reported and assessed in report [#3557062](https://hackerone.com/reports/3557062). Both reports describe a business logic vulnerability where free-tier users can bypass Pro plan subscription restrictions by manipulating the access\_level parameter to "read" when creating project invitation links via the POST /projects/{project\_id}/magic-codes endpoint.

The original report was submitted 20 days prior and documented the same technical vulnerability, including the identical API endpoint, parameter manipulation technique, and bypass outcome. As a result, this report is being closed as a duplicate.

We appreciate your effort in testing this program's security and encourage you to continue reporting any new vulnerabilities you discover.

Thanks,
[@h1\_analyst\_mars](https://hackerone.com/h1_analyst_mars)

[h1\_analyst\_mars](https://hackerone.com/h1_analyst_mars)

HackerOne triage

**reopened** this report.

[March 11, 2026, 1:53pm UTC](https://hackerone.com/reports/3591764#activity-39978565)

[![ziad momen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/cbcbf98f1b109b2be47f5597756b8bb14b56c3ba66c64b2d2db1e108bfcc0a32)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

posted a comment.

[March 11, 2026, 2:37pm UTC](https://hackerone.com/reports/3591764#activity-39979376)

MenuMenu

Hi [@h1\_analyst\_mars](https://hackerone.com/h1_analyst_mars) , hope your are doing great!

Are there any updates regarding my report?

[![ziad momen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/cbcbf98f1b109b2be47f5597756b8bb14b56c3ba66c64b2d2db1e108bfcc0a32)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

posted a comment.

[March 13, 2026, 5:16pm UTC](https://hackerone.com/reports/3591764#activity-40035360)

MenuMenu

Any Updates? I hope to hear good news!

[![](https://profile-photos.hackerone-user-content.com/variants/dh7eyfnor7buii9e32tcuvi4crfp/2a499a0b732abf809ef45635d9ad400431480fafd17452e700cbdba6fd64fe85)](https://hackerone.com/h1_analyst_mars)

[h1\_analyst\_mars](https://hackerone.com/h1_analyst_mars)

HackerOne triage

changed the status to **Needs more info**.

[March 16, 2026, 7:16am UTC](https://hackerone.com/reports/3591764#activity-40069615)

MenuMenu

Hi [@ziadmomen](https://hackerone.com/ziadmomen),

We are getting the following response which is supposed to be the proper response.

imageimage

**Image**•740.56 KiB•F5542583: Screenshot\_2026-03-16\_at\_12.45.20\_PM.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

Thanks,
[@h1\_analyst\_mars](https://hackerone.com/h1_analyst_mars)

attachmentsattachments

**1 attachment**

CollapseCollapse

- F5542583: [Screenshot\_2026-03-16\_at\_12.45.20\_PM.png](ATTACHMENT_URL_REDACTED)


[![Kiiw](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/lovable_kiiw)

[lovable\_kiiw](https://hackerone.com/lovable_kiiw)

Lovable VDP staff

updated the severity from
high(8.8)
to low.

[March 16, 2026, 11:14am UTC](https://hackerone.com/reports/3591764#activity-40074850)

MenuMenu

Free use of paid feature. No security impact.

[![Kiiw](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/lovable_kiiw)

[lovable\_kiiw](https://hackerone.com/lovable_kiiw)

Lovable VDP staff

closed the report and changed the status to **Resolved**.

Updated [June 12, 2026, 9:40am UTC](https://hackerone.com/reports/3591764#activity-40074887)

MenuMenu

Hi [@ziadmomen](https://hackerone.com/ziadmomen),

Thank you for your report and for your efforts to keep our users safe!

This issue has now been resolved. We look forward to receiving more reports from you in the future.

Best regards,
████████

[![ziad momen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/cbcbf98f1b109b2be47f5597756b8bb14b56c3ba66c64b2d2db1e108bfcc0a32)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

posted a comment.

Updated [June 12, 2026, 9:40am UTC](https://hackerone.com/reports/3591764#activity-40075128)

MenuMenu

Hi [@lovable\_](https://hackerone.com/lovable_) ████████ hope you are doing great!

May you recheck the severity. At least it should be medium not low.As there is a vulnerability similar to this I have reported closed as a duplicate for medium severity

[![ziad momen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/cbcbf98f1b109b2be47f5597756b8bb14b56c3ba66c64b2d2db1e108bfcc0a32)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

posted a comment.

Updated [March 16, 2026, 11:30am UTC](https://hackerone.com/reports/3591764#activity-40075184)

MenuMenu

[#3370430](https://hackerone.com/reports/3370430)
this one also has a similar idea bypassing the plane which is closed as medium

[![Kiiw](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/lovable_kiiw)

[lovable\_kiiw](https://hackerone.com/lovable_kiiw)

Lovable VDP staff

updated the severity from low to medium.

[March 16, 2026, 11:44am UTC](https://hackerone.com/reports/3591764#activity-40075421)

MenuMenu

Free use of paid feature. No security impact.

[![ziad momen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/cbcbf98f1b109b2be47f5597756b8bb14b56c3ba66c64b2d2db1e108bfcc0a32)\\
\\
ID-verifiedHacker that has successfully completed an ID verification check.](https://hackerone.com/ziadmomen)

[ziadmomen](https://hackerone.com/ziadmomen)

posted a comment.

Updated [June 12, 2026, 9:40am UTC](https://hackerone.com/reports/3591764#activity-40075624)

MenuMenu

Thank you so much [@lovable\_](https://hackerone.com/lovable_) ████████
Can we disclose this report?

[ziadmomen](https://hackerone.com/ziadmomen)

requested to disclose this report.

[March 16, 2026, 11:58am UTC](https://hackerone.com/reports/3591764#activity-40075693)

[lovable\_kiiw](https://hackerone.com/lovable_kiiw)

Lovable VDP staff

agreed to disclose this report.

[March 16, 2026, 12pm UTC](https://hackerone.com/reports/3591764#activity-40075750)

This report has been disclosed.

[March 16, 2026, 12pm UTC](https://hackerone.com/reports/3591764#activity-40075751)

Reported on

March 8, 2026, 8:08am UTC

Reported by

![ziadmomen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/7d6e0417f1813b18bcb08485a78f3f450b5240c7e7ad6e0c3ca1ab23e56369d2)

[ziadmomen](https://hackerone.com/ziadmomen)

Reported to

**[Lovable VDP](https://hackerone.com/lovable-vdp)**

Managed

Participants

[![ziadmomen](https://profile-photos.hackerone-user-content.com/variants/yf4ax5bn1wwfe0jwc0ilxszcssjv/7d6e0417f1813b18bcb08485a78f3f450b5240c7e7ad6e0c3ca1ab23e56369d2)](https://hackerone.com/ziadmomen)

[![lovable_kiiw](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/lovable_kiiw)

[![h1_analyst_diva](https://profile-photos.hackerone-user-content.com/variants/k5s1d31x68des504elg1u815wli5/dd1dfc8fc7352772e4a8994c07e1d628d34edc74db48377c707f10829dc23d53)](https://hackerone.com/h1_analyst_diva)

[![h1_analyst_mars](https://profile-photos.hackerone-user-content.com/variants/dh7eyfnor7buii9e32tcuvi4crfp/dd1dfc8fc7352772e4a8994c07e1d628d34edc74db48377c707f10829dc23d53)](https://hackerone.com/h1_analyst_mars)

Report Id

[#3591764](https://hackerone.com/reports/3591764)

Resolved

Severity

Medium (4 ~ 6.9)

* * *

Disclosed

March 16, 2026, 12pm UTC

Weakness

Business Logic Errors

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

## Validation Agent Reasoning

CloseClose

Disagree with the reasoning?Let us know

## Exploit Agent Reasoning

CloseClose

## Linear Agent Reasoning

CloseClose

StripeM-Inner