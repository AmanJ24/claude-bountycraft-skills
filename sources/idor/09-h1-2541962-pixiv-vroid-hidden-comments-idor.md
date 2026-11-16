---
title: "Disclose Hidden Comments on Media Section of hub.vroid.com"
author: giwadaoud
url: https://hackerone.com/reports/2541962
date_published: 2024-06-07
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/2541962
bounty: $500 (+$50 retest)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# Disclose Hidden Comments on Media Section of hub.vroid.com

Reported to: pixiv | Reporter: giwadaoud | Reported: 2024-06-07 | Disclosed: 2026-01-18 | State: Resolved | Severity: medium

Timeline

[giwadaoud](https://hackerone.com/giwadaoud)

submitted a report to [**pixiv**](https://hackerone.com/pixiv).

June 7, 2024, 10:30pm UTC

I discovered a vulnerability related to hidden comments in the Media section on [https://hub.vroid.com.](https://hub.vroid.com./) Initially, a user allowed comments on their media post but later disable them, making all comments invisible to everyone except the admin. However, by intercepting a request where a user can like any comment with a specific ID, I found that it is possible to like hidden comments by inputting their IDs. The response from the server not only confirms the like action but also reveals the content of the hidden comment, which should only be visible to the original poster of this media. This vulnerability in the endpoint allows unauthorized disclosure of hidden comments on any posts in the Media section.

**Image**•9.09 KiB•F3340195: image.png



## Steps To Reproduce:

**UserAttacker** **UserVictim**

1. Create two random accounts on [https://accounts.pixiv.net/](https://accounts.pixiv.net/)
2. Login then both on [https://hub.vroid.com](https://hub.vroid.com/)
3. From UserVictim navigate to **submit character** and submit a character then scroll down you will see:

**Image**•2.04 KiB•F3340256: image.png



4. Upload media here and allow comments
5. Post any random comment then disable comments section on the media of the UserVictim
6. From UserVictim we need to get the comment ID to use it later (this step only for testing, real attack dont need this step), for that, click ctrl + U to view the page source then ctrl + F and search for **entityIds**

**Image**•2.35 KiB•F3340234: image.png



7. Save the comment ID to use it later.
8. From UserAttacker, intercept any POST request made to [https://hub.vroid.com/](https://hub.vroid.com/) and send it to the repeater
9. Update the POST request URL and parameter to the below:

**Code**•293 Bytes

1POST /api/statuses/PASTE\_ID\_HERE/hearts HTTP/2
2Host: hub.vroid.com
3Cookie: ATTACKER\_COOKIES
4Sec-Ch-Ua: "Opera";v="109", "Not:A-Brand";v="8", "Chromium";v="123"
5Accept: application/json, text/plain, \*/\*
6Content-Type: application/json
7X-Api-Version: 11
8Sec-Ch-Ua-Mobile: ?0
9Content-Length: 2
1011{}

10. Send the request, and you will find the content of the comment and its owner in the response. In this case, the attacker will be able to disclose any hidden comment.

**Image**•19.66 KiB•F3340305: image.png



## Impact

This vulnerability allows an attacker to disclose hidden comments of any user due to a vulnerable endpoint. These hidden comments should only be visible to the admin.


**4 attachments**

- F3340195: [image.png]

- F3340234: [image.png]

- F3340256: [image.png]

- F3340305: [image.png]

Show older activities

[h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn)

HackerOne triage

changed the status to **Needs more info**.

[June 24, 2024, 9:18pm UTC](https://hackerone.com/reports/2541962#activity-28193005)

Hi [@giwadaoud](https://hackerone.com/giwadaoud),

It appears it was an issue with Firefox. I was able to upload the VRM file via Chrome.

Can you please elaborate on the following steps:

- Upload media here and allow comments
- Post any random comment then disable comments section on the media of the UserVictim

Where is this located? It says my character is private.

Kind regards,
[@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn)

[giwadaoud](https://hackerone.com/giwadaoud)

changed the status to **New**.

[June 25, 2024, 1:21pm UTC](https://hackerone.com/reports/2541962#activity-28203567)

Hi [@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn),

Sorry for the inconveniences. Please find below a PoC video that shows all the steps required:
[https://www.youtube.com/watch?v=pPTG7s2PEUI&ab\_channel=GiwaDaoud](https://www.youtube.com/watch?v=pPTG7s2PEUI&ab_channel=GiwaDaoud)

Please let me know if you require any additional information.

Kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn)

HackerOne triage

changed the status to **Needs more info**.

[June 28, 2024, 6:41pm UTC](https://hackerone.com/reports/2541962#activity-28274747)

Hi [@giwadaoud](https://hackerone.com/giwadaoud),

Please immediately remove this video from youtube as that violates the code of conduct of H1. Any videos should be uploaded directly to the report as an attachment.

Thank you,
[@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn)

[giwadaoud](https://hackerone.com/giwadaoud)

changed the status to **New**.

[June 28, 2024, 7:09pm UTC](https://hackerone.com/reports/2541962#activity-28275413)

Hi [@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn),

I'm sorry for the inconvenience. The video was uploaded as unlisted on YouTube, and I had no idea that this would violate the code of conduct of H1. I have removed it from YouTube and apologize again. Please find the attached PoC: [2024-06-25\_14-34-29.mkv (F3396850)]

kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)


**1 attachment**

- F3396850: [2024-06-25\_14-34-29.mkv]

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[July 2, 2024, 8:02am UTC](https://hackerone.com/reports/2541962#activity-28311455)

Hi [@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn),

Is there any update regarding this?

kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn)

HackerOne triage

changed the status to **Needs more info**.

Updated [July 2, 2024, 10:05pm UTC](https://hackerone.com/reports/2541962#activity-28322871)

Hi [@giwadaoud](https://hackerone.com/giwadaoud),

The video you uploaded is extremely laggy and it's not possible to follow along. Can you please provide either a smooth PoC video or written instructions supplemented with screenshots as necessary to address the following points:

- Upload media here and allow comments
- Post any random comment then disable comments section on the media of the UserVictim

Thank you,
[@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn)

[giwadaoud](https://hackerone.com/giwadaoud)

changed the status to **New**.

[July 3, 2024, 6:15pm UTC](https://hackerone.com/reports/2541962#activity-28335733)

Hi [@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn),

Please find below a PoC video that shows all the steps required:

videovideo

**Video**•34.62 MiB•F3408980: 2024-07-03\_19-28-58.mp4

I hope it will be clear this time.

Best regards,
[@giwadaoud](https://hackerone.com/giwadaoud)


**1 attachment**

- F3408980: [2024-07-03\_19-28-58.mp4]

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[July 9, 2024, 1:01pm UTC](https://hackerone.com/reports/2541962#activity-28405071)

Hi [@h1\_analyst\_bjorn](https://hackerone.com/h1_analyst_bjorn),

Is there any update regarding this?

kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[h1\_analyst\_charlie](https://hackerone.com/h1_analyst_charlie)

HackerOne triage

changed the report title from **Disclose hidden comments on Media Section** to **Disclose Hidden Comments on Media Section of hub.vroid.com**.

[July 9, 2024, 9:23pm UTC](https://hackerone.com/reports/2541962#activity-28417453)

[h1\_analyst\_charlie](https://hackerone.com/h1_analyst_charlie)

HackerOne triage

updated the severity from medium to
medium(4.6)
.

[July 9, 2024, 9:23pm UTC](https://hackerone.com/reports/2541962#activity-28417454)

[h1\_analyst\_charlie](https://hackerone.com/h1_analyst_charlie)

HackerOne triage

changed the status to **Triaged**.

[July 9, 2024, 9:23pm UTC](https://hackerone.com/reports/2541962#activity-28417460)

Hello [@giwadaoud](https://hackerone.com/giwadaoud),

Thank you for your submission! We were able to validate your report, and have submitted it to the appropriate remediation team for review. They will let us know the final ruling on this report, and when/if a fix will be implemented. Please note that the status and severity are subject to change.

Thanks,
[@h1\_analyst\_charlie](https://hackerone.com/h1_analyst_charlie)

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[July 15, 2024, 8:19pm UTC](https://hackerone.com/reports/2541962#activity-28525642)

Hi [@h1\_analyst\_charlie](https://hackerone.com/h1_analyst_charlie),

Any updates?

Kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[pixiv](https://hackerone.com/pixiv)

rewarded [giwadaoud](https://hackerone.com/giwadaoud) with a **$500** bounty.

[July 16, 2024, 8:34am UTC](https://hackerone.com/reports/2541962#activity-28533097)

Hi [@giwadaoud](https://hackerone.com/giwadaoud),

Thanks for reporting!
We were able to reproduce this issue, and have confirmed that this is a valid vulnerability.
Based on our bounty table, we would like to issue you a bounty of $500.

We'll get back to you when the issue has been resolved.
Thanks,

[sue445](https://hackerone.com/sue445)

pixiv staff

changed the status to **Retesting**

The reporter has been invited to perform a retest for **$50**.

Updated [July 25, 2024, 8:08am UTC](https://hackerone.com/reports/2541962#activity-28724941)

We have fixed the problem of the issue.
Could you retest that this problem does not happen?
Thanks,

[giwadaoud](https://hackerone.com/giwadaoud)

completed a retest.

[July 25, 2024, 8:57pm UTC](https://hackerone.com/reports/2541962#activity-28738382)

**Retest finding result**

- Are you able to reproduce the vulnerability report?

**No, the fix works.**

- Please provide a short summary with the results of your retest

Description of the Original Vulnerability:
A vulnerability allowed liking hidden comments by their IDs, revealing the comment content in the server response.

Retesting Steps:

1. Reproduction of Original Steps:

  - From UserVictim: Submitted a media post, allowed comments, posted a comment, then disabled comments. Retrieved the comment ID.
  - From UserAttacker: Intercepted a POST request and updated it to like the hidden comment using the retrieved ID.

2. Validation of the Fix:

  - Sent the updated POST request and confirmed the server now returns "COMMON\_NOT\_FOUND" instead of disclosing hidden comment content.

3. Testing for Variations:

  - Tested related endpoints to confirm no similar vulnerabilities.

Outcome:

Confirmed the vulnerability is fixed and hidden comments are no longer disclosed. The server returns "COMMON\_NOT\_FOUND" for attempts to interact with hidden comments.


**1 attachment**

- F3468745: [Screenshot\_2024-07-25\_235110.png]

[pixiv](https://hackerone.com/pixiv)

awarded $50 to [giwadaoud](https://hackerone.com/giwadaoud) for completing the retest.

[July 26, 2024, 6:37am UTC](https://hackerone.com/reports/2541962#activity-28742855)

thx!

[sue445](https://hackerone.com/sue445)

pixiv staff

closed the report and changed the status to **Resolved**.

[July 26, 2024, 6:37am UTC](https://hackerone.com/reports/2541962#activity-28742857)

[giwadaoud](https://hackerone.com/giwadaoud)

requested to disclose this report.

[July 26, 2024, 10:42am UTC](https://hackerone.com/reports/2541962#activity-28749011)

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[July 26, 2024, 11:24am UTC](https://hackerone.com/reports/2541962#activity-28750762)

Hi [@sue445](https://hackerone.com/sue445),

I would like to ask for your permission to write and publish a detailed write-up about this vulnerability.

The write-up would include the nature of the vulnerability, the steps to reproduce it, its impact, and the fix, while ensuring no sensitive information or details that could compromise security are shared.

Could you please let me know if this would be acceptable?

Thank you for your consideration.

Best regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[July 30, 2024, 11:44am UTC](https://hackerone.com/reports/2541962#activity-28806597)

Hi [@sue445](https://hackerone.com/sue445) ,

Is there any updates regarding this?

Kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[August 3, 2024, 9:43am UTC](https://hackerone.com/reports/2541962#activity-28884492)

Hi [@sue445](https://hackerone.com/sue445),

I hope this message finds you well. I wanted to follow up on my previous requests regarding permission to write and publish a detailed write-up about the vulnerability I reported. It's been over a week since my initial request, and I haven't received a response yet.

The write-up would include the nature of the vulnerability, the steps to reproduce it, its impact, and the fix, while ensuring no sensitive information or details that could compromise security are shared. I've already prepared a draft, and I'm eager to share this learning experience with the community.

Could you please let me know if this would be acceptable?

Thank you for your assistance and understanding. I look forward to your response.

Best regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[sue445](https://hackerone.com/sue445)

pixiv staff

posted a comment.

[August 6, 2024, 6:09am UTC](https://hackerone.com/reports/2541962#activity-28923152)

[@giwadaoud](https://hackerone.com/giwadaoud) Sorry for the late reply.

After discussing the issue internally, I received a response that this would not be a problem to publish the article.

Could you please give us the URL after you publish the article?

[giwadaoud](https://hackerone.com/giwadaoud)

posted a comment.

[August 6, 2024, 9:35am UTC](https://hackerone.com/reports/2541962#activity-28927301)

Hi [@sue445](https://hackerone.com/sue445),

Here is the URL of the article I published:
[https://medium.com/@daoudgiwa/a-journey-to-my-first-bug-bounty-0f74f139c845](https://medium.com/@daoudgiwa/a-journey-to-my-first-bug-bounty-0f74f139c845)

Kind regards,
[@giwadaoud](https://hackerone.com/giwadaoud)

[namazu](https://hackerone.com/namazu)

pixiv staff

agreed to disclose this report.

[January 18, 2026, 11:24am UTC](https://hackerone.com/reports/2541962#activity-39022733)

This report has been disclosed.

[January 18, 2026, 11:24am UTC](https://hackerone.com/reports/2541962#activity-39022734)

Reported on

June 7, 2024, 10:30pm UTC

Reported by



[giwadaoud](https://hackerone.com/giwadaoud)

Reported to

**[pixiv](https://hackerone.com/pixiv)**

Managed

Participants

Report Id

[#2541962](https://hackerone.com/reports/2541962)

Resolved

Severity

Medium (4.6)

* * *

Disclosed

January 18, 2026, 11:24am UTC

Weakness

Insecure Direct Object Reference (IDOR)

CVE ID

None

* * *

Bounty

$500

Retest

None

Total reward

None

* * *

Account details

None

* * *

StripeM-Inner
