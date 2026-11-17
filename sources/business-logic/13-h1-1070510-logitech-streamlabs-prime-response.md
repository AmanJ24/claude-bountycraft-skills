---
title: "Manipulating response leads to free access to Streamlabs Prime"
author: sudi
url: https://hackerone.com/reports/1070510
date_published: 2021-01-03
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1070510
bounty: not stated (amount hidden on page; reporter disputed bounty amount in thread)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

80

[#1070510](https://hackerone.com/reports/1070510)

Copy report idCopy report id

Manipulating response leads to free access to Streamlabs Prime

- Share:

Report

Timeline

[![sudi](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)\\
\\
Clear-verified badgeHacker that has completed both ID-verification and a background check.](https://hackerone.com/sudi)

[sudi](https://hackerone.com/sudi)

submitted a report to [**Logitech**](https://hackerone.com/logitech).

January 3, 2021, 10:04am UTC

MenuMenu

Heyy team,
I have a found cool bug which allows me to get access to streamlabs prime features for free.

Here is the api endpoint which checks whether the user has a prime subscription or not:

[https://streamlabs.com/api/v5/user/prime/subscription](https://streamlabs.com/api/v5/user/prime/subscription)

CodeCode

**Code**•315 Bytes

Auto-formatAuto-format

Unwrap linesUnwrap lines

DownloadDownload

CopyCopy

CollapseCollapse

1{2"is\_active":false,3"is\_pending":false,4"is\_past\_due":false,5"is\_canceled":false,6"is\_pro":false,7"type":"",8"free\_trial\_claimed":false,9"hibernate\_claimed":false,10"free\_trial\_eligible":false,11"hibernate\_eligible":false,12"cooldown\_eligible":false,13"cooldown\_seconds\_past":null14}

Everything was set to false, so I decided to use the Burp's _Match and Replace_ rule to replace every ocurrence of _false_ in the Response body to _true_.
And as expected I got access to some of the prime features, the most interesting one was the streamlabs All Stars rewards, which is available at this url, [https://streamlabs.com/dashboard#/streamlabs-rewards?skipPrimeOnboarding=1](https://streamlabs.com/dashboard#/streamlabs-rewards?skipPrimeOnboarding=1)

When a user which doesn't have the Streamlabs Prime tries to access the streamlabs-reward endpoint the _reedeemed_ option won't be there:

imageimage

**Image**•152.86 KiB•F1142675: XmrdC7NQ1V.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

Now after setting the match and replace in burp now if we try to visit the streamlabs-reward endpoint.

imageimage

**Image**•133.41 KiB•F1142677: 1K1Ow4XZhv.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

To confirm that I can actually redeem the reward I went ahead and clicked on the redeem button then it asked for en email and I used my hackerone alias email. Just in few seconds a mail arrived in my inbox with the _code_ which I can use to get the Logitech mouse for free

imageimage

**Image**•221.05 KiB•F1142679: kB9rNTOYcg.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

imageimage

**Image**•51.98 KiB•F1142680: M24GT6WyfK.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

I checked other features too like Multistreaming, I was able to get the _RTMP url_ and _Stream key_ [https://streamlabs.com/dashboard#/multistream/settings](https://streamlabs.com/dashboard#/multistream/settings)

Without match and replace rule set:

imageimage

**Image**•52.39 KiB•F1142690: 045dggyVXm.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

After setting match and replace rule:

imageimage

**Image**•45.36 KiB•F1142698: In1bDyLVZV.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

* * *

**Steps to Reproduce:**

In your burpsuite _Match and Replace_ rule add this two entries:

CodeCode

**Code**•51 Bytes

Unwrap linesUnwrap lines

DownloadDownload

CopyCopy

CollapseCollapse

11st.
2Type: Response Body
3Match: false
4Replace: true

CodeCode

**Code**•71 Bytes

Unwrap linesUnwrap lines

DownloadDownload

CopyCopy

CollapseCollapse

12nd.
2Type: Response Body
3Match: "blocked":true
4Replace: "blocked":false

Similar to this in the screenshot

imageimage

**Image**•6.57 KiB•F1142700: dRdjCNDNX4.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

1. Now goto [https://streamlabs.com/dashboard#/streamlabs-rewards?skipPrimeOnboarding=1](https://streamlabs.com/dashboard#/streamlabs-rewards?skipPrimeOnboarding=1)
2. The redeem button should be there, as I have shown above in the screenshots [1K1Ow4XZhv.png (F1142677)](ATTACHMENT_URL_REDACTED)
3. Click on the button , provide your email and you should get the coupon code

Similar steps for checking the multistreaming endpoint, [https://streamlabs.com/dashboard#/multistream/settings](https://streamlabs.com/dashboard#/multistream/settings)

**Note:** It might happen that when you load this url: [https://streamlabs.com/dashboard#/streamlabs-rewards?skipPrimeOnboarding=1](https://streamlabs.com/dashboard#/streamlabs-rewards?skipPrimeOnboarding=1), this page might appear where it asks you to connect your paypal acc:

imageimage

**Image**•39.69 KiB•F1142703: firefox\_KZCyY6varS.png

Open image in fullOpen image in full

[Download mediaDownload media](ATTACHMENT_URL_REDACTED)

Copy reference IDCopy reference ID

CollapseCollapse

![](ATTACHMENT_URL_REDACTED)

In that case , provide any fake email id and click on continue and also skip the other options also.Once it done now try to open the streamlab-rewards url in a new tab , if it still doesn't works try loading this url instead [https://streamlabs.com/dashboard#/streamlabs-rewards](https://streamlabs.com/dashboard#/streamlabs-rewards) (here I have removed the parameter), keep refreshing the page until you get the right one.

* * *

## Impact

An attacker can exploit this to get free access to prime features and also get free items at no cost as many as he wants .

**PS:** If you don't mind can I use that coupon code, for getting that sweet gaming mouse for free (it looks really cool lol) , no worries if your answer is no :)

Thankyou
Regards
Sudhanshu

attachmentsattachments

**8 attachments**

CollapseCollapse

- F1142675: [XmrdC7NQ1V.png](ATTACHMENT_URL_REDACTED)

- F1142677: [1K1Ow4XZhv.png](ATTACHMENT_URL_REDACTED)

- F1142679: [kB9rNTOYcg.png](ATTACHMENT_URL_REDACTED)

- F1142680: [M24GT6WyfK.png](ATTACHMENT_URL_REDACTED)

- F1142690: [045dggyVXm.png](ATTACHMENT_URL_REDACTED)

- F1142698: [In1bDyLVZV.png](ATTACHMENT_URL_REDACTED)

- F1142700: [dRdjCNDNX4.png](ATTACHMENT_URL_REDACTED)

- F1142703: [firefox\_KZCyY6varS.png](ATTACHMENT_URL_REDACTED)


[![](https://profile-photos.hackerone-user-content.com/variants/2bkb69nvrwhkybw7m3gi9o2dumgp/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)](https://hackerone.com/h1_analyst_caesar)

[h1\_analyst\_caesar](https://hackerone.com/h1_analyst_caesar)

closed the report and changed the status to **Duplicate** ( [#1005330](https://hackerone.com/reports/1005330)).

[January 4, 2021, 8:30am UTC](https://hackerone.com/reports/1070510#activity-10259746)

MenuMenu

Hi [@sudi](https://hackerone.com/sudi),

Unfortunately, this was submitted previously by another researcher, but we appreciate your work and look forward to additional reports from you.

For transparency, we have invited you to the original report. Please do not comment on the original submission. If you have any further questions or concerns, please post it on this report instead.

Have a great day ahead!

Best regards,
[@turtle\_shell](https://hackerone.com/turtle_shell)

[![Sudhanshu Rajbhar](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)\\
\\
Clear-verified badgeHacker that has completed both ID-verification and a background check.](https://hackerone.com/sudi)

[sudi](https://hackerone.com/sudi)

posted a comment.

[January 4, 2021, 8:58am UTC](https://hackerone.com/reports/1070510#activity-10260258)

MenuMenu

Hey [@turtle\_shell](https://hackerone.com/turtle_shell),
I see that this bug has been reported already but you should have look at the impact here in my report is different than what was showed in this report [#1005330](https://hackerone.com/reports/1005330), the researcher showed that he can get access to the _Custom Twitch Bot Name_ option and later the logitech team staff commented out that :
[@logi\_salman](https://hackerone.com/logi_salman)

> You cannot use custom bot name or any other prime features by going through this flow.

You can see clearly in my report that I am able to get access other features too like multistreaming, streamlabs All Stars rewards,etc

The All start rewards allows me to get **as many gifts as I want** at **$0.0** cost.
for eg: By creating 1000 accounts I can claim 1000 gifts , the gift is a logitech gaming mice which cost on average is around $30.
So 1000 logitech gaming mouse costs would be 30\*1000 = **$30,000** . I can get all of this for **free**.

I have already shown in my mail where I recieved coupon codes for the logitech gaming mice.

Some new features have been added to the _Streamlabs Prime_ , when the other researcher reported the issue there wasn't any impact I believe. But if you take a look at my report you can see this impact the company financially.
I have just shown eg for $30,000 this can go even far beyond that.

Kindly have a look at this report again and also allow the Logitech's team to look into this.

If any more info is required , feel free to ask me . I am happy to assist.

Thankyou
Regards
Sudhanshu

[![](https://profile-photos.hackerone-user-content.com/variants/2bkb69nvrwhkybw7m3gi9o2dumgp/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)](https://hackerone.com/h1_analyst_caesar)

[h1\_analyst\_caesar](https://hackerone.com/h1_analyst_caesar)

**reopened** this report.

[January 4, 2021, 10:51am UTC](https://hackerone.com/reports/1070510#activity-10261325)

MenuMenu

I'll take another look into it - I was also not entirely sure about closing it as a Duplicate. Thanks for the additional information

[![](https://profile-photos.hackerone-user-content.com/variants/2bkb69nvrwhkybw7m3gi9o2dumgp/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)](https://hackerone.com/h1_analyst_caesar)

[h1\_analyst\_caesar](https://hackerone.com/h1_analyst_caesar)

changed the status to **Triaged**.

[January 5, 2021, 1:11pm UTC](https://hackerone.com/reports/1070510#activity-10274627)

MenuMenu

Hello [@sudi](https://hackerone.com/sudi),

Thank you for your submission! We were able to validate your report, and have submitted it to the appropriate remediation team for review. They will let us know the final ruling on this report, and when/if a fix will be implemented. Please note that the status and severity are subject to change.

Regards,
[@turtle\_shell](https://hackerone.com/turtle_shell)

[h1\_analyst\_caesar](https://hackerone.com/h1_analyst_caesar)

added weakness "Business Logic Errors" and removed weakness "Improper Check or Handling of Exceptional Conditions".

[January 5, 2021, 1:16pm UTC](https://hackerone.com/reports/1070510#activity-10274672)

[h1\_analyst\_caesar](https://hackerone.com/h1_analyst_caesar)

updated the severity from critical to
medium(6.5)
.

[January 5, 2021, 1:16pm UTC](https://hackerone.com/reports/1070510#activity-10274674)

[logi\_aron](https://hackerone.com/logi_aron)

updated the severity from
medium(6.5)
to
medium(4.3)
.

[January 5, 2021, 4:43pm UTC](https://hackerone.com/reports/1070510#activity-10276808)

[Logitech](https://hackerone.com/logitech)

rewarded [sudi](https://hackerone.com/sudi) with a bounty.

[January 5, 2021, 4:44pm UTC](https://hackerone.com/reports/1070510#activity-10276818)

[![Sudhanshu Rajbhar](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)\\
\\
Clear-verified badgeHacker that has completed both ID-verification and a background check.](https://hackerone.com/sudi)

[sudi](https://hackerone.com/sudi)

posted a comment.

[January 5, 2021, 4:48pm UTC](https://hackerone.com/reports/1070510#activity-10276856)

MenuMenu

Hey team,
I disagree with the bounty amount , regarding as this bug would have allowed any user to get items for free this could lead to a financial loss to the company and also I have shown that i could get access to the other Prime features too.

Can you reconsider the bounty amount please?

Thankyou

[![Aron Rosenberg](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/logi_aron)

[logi\_aron](https://hackerone.com/logi_aron)

posted a comment.

[January 13, 2021, 1:03am UTC](https://hackerone.com/reports/1070510#activity-10346517)

MenuMenu

Hi [@sudi](https://hackerone.com/sudi) We score all issues per the CVSS system which classifies this as a Medium severity.

[![](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/logi_harsha)

[logi\_harsha](https://hackerone.com/logi_harsha)

closed the report and changed the status to **Resolved**.

[January 13, 2021, 1:04am UTC](https://hackerone.com/reports/1070510#activity-10346521)

MenuMenu

hi [@sudi](https://hackerone.com/sudi) this issue is fixed now, please let us know if you are still able to reproduce this.

Thanks

[![Sudhanshu Rajbhar](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)\\
\\
Clear-verified badgeHacker that has completed both ID-verification and a background check.](https://hackerone.com/sudi)

[sudi](https://hackerone.com/sudi)

requested to disclose this report.

[January 13, 2021, 1:06pm UTC](https://hackerone.com/reports/1070510#activity-10350899)

MenuMenu

Hey [@logi\_smaddela](https://hackerone.com/logi_smaddela),
I can confirm the fix, now if I try to redeem a free gift by following the same way a `403 forbidden error` is received.
Btw I can still get _RTMP url and Stream key_ from the multistreaming endpoint.

If it's intended then kindly accept the disclosure request.

Regards
Sudhanshu

[![Sudhanshu Rajbhar](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/05370d5711f52cb436358cc33184ae5c3ed8ed29e76e71bd91e66faa9bd6e9ea)\\
\\
Clear-verified badgeHacker that has completed both ID-verification and a background check.](https://hackerone.com/sudi)

[sudi](https://hackerone.com/sudi)

posted a comment.

[January 21, 2021, 3:34am UTC](https://hackerone.com/reports/1070510#activity-10430682)

MenuMenu

Hey team,
If everything this fine is ok , can we disclose this?

[logi\_harsha](https://hackerone.com/logi_harsha)

agreed to disclose this report.

[January 21, 2021, 3:53am UTC](https://hackerone.com/reports/1070510#activity-10430727)

This report has been disclosed.

[January 21, 2021, 3:53am UTC](https://hackerone.com/reports/1070510#activity-10430728)

Reported on

January 3, 2021, 10:04am UTC

Reported by

![sudi](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)

[sudi](https://hackerone.com/sudi)

Reported to

**[Logitech](https://hackerone.com/logitech)**

Managed

Participants

[![sudi](https://profile-photos.hackerone-user-content.com/variants/h54zoekbmc0kqlq42xu2vqj6ge13/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)](https://hackerone.com/sudi)

[![logi_harsha](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/logi_harsha)

[![logi_aron](https://hackerone.com/assets/avatars/default-14ffa99f59cd01423c64904352cc130ffcb6a802eadfd11777a54485749e60f2.png)](https://hackerone.com/logi_aron)

[![h1_analyst_caesar](https://profile-photos.hackerone-user-content.com/variants/2bkb69nvrwhkybw7m3gi9o2dumgp/f4770e29726b679d8210b3bcebb50139392345e8ead50e7d67de79a93076e327)](https://hackerone.com/h1_analyst_caesar)

Report Id

[#1070510](https://hackerone.com/reports/1070510)

Resolved

Severity

Medium (4.3)

* * *

Disclosed

January 21, 2021, 3:53am UTC

Weakness

Business Logic Errors

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

StripeM-Inner

## Validation Agent Reasoning

CloseClose

Disagree with the reasoning?Let us know

## Exploit Agent Reasoning

CloseClose

## Linear Agent Reasoning

CloseClose