---
title: "Race Condition leads to undeletable group member"
author: yashrs (Yash Sodha)
url: https://hackerone.com/reports/604534
date_published: 2019-06-09
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/604534
bounty: not stated (bounty awarded by HackerOne on 2019-08-16, amount hidden on report)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Scrape status: FULL (complete report body and timeline rendered, including "Show older activities" entries; bounty amount hidden by HackerOne). Presigned attachment URLs replaced by [REDACTED]. Invite token, CSRF token and cookies in the sample request replaced with [REDACTED]. Code block reconstructed from rendered output. One attachment was redacted by HackerOne staff at the researcher's request before disclosure.

Report #604534: Race Condition leads to undeletable group member
Reported to: HackerOne (security program, target: ctf.hacker101.com). Reported by: yashrs. Reported on: June 9, 2019, 10:26am UTC. Disclosed: March 20, 2020, 5:17pm UTC.
Status: Resolved. Severity: Low (2.4). Weakness: Time-of-check Time-of-use (TOCTOU) Race Condition. CVE: None. Bounty / Total reward: Hidden.

## Summary by yashrs

It was a small race condition bug in which a group user couldn't be removed from the group even by the admin after they join. This was kinda a fun bug(though low severity) to find when I was learning about race conditions way back in June last year. I thought about trying this after seeing another disclosed report on RC on ctf.hacker101.com itself

## Report body (yashrs, June 9, 2019, 10:26am UTC)

Hi,

**Summary:**
There exists a Race Condition in which the user can add themselves twice to a group which will make them unremovable from group. They themselves cannot remove themselves from the group as well as the group leader cannot remove that user from the group. Ofcourse this is a low severity as that malicious user first needs to have a valid invitation link, but nevertheless this leads to permanent membeship of group so this needs to be fixed in my opinion.

### Steps To Reproduce

We need two accounts to reproduce this.

- First account will be leader account

- Second account will be a normal account.

- Create a group from the first account and create an invitation link.

- Join using the invitation link on the second account and intercept the request:

```
POST /group/post_join HTTP/1.1
Host: ctf.hacker101.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://ctf.hacker101.com/group/join?invite=[REDACTED]
Content-Type: application/x-www-form-urlencoded
Content-Length: 109
Connection: close
Cookie: COOKIES
Upgrade-Insecure-Requests: 1

csrf=[REDACTED]&invite=[REDACTED]
```

Repeat the above requests in parallel. I did it for 5 parallel requests and I was able to add myself 2 times to that group:

Image F505223: Capture.PNG [REDACTED presigned URL]

- Now try deleting the user from first account(group leader). No matter how much you try, you won't be able to delete it.

This would lead to a state where the group leader will have to delete the group and create a new group if he wants the malicious user to get removed.

Thanks,
Yash

## Impact

Irremovable/permanent member of the group. Even the group leader cannot remove that user.

## Timeline

- **yashrs** (June 9, 2019, 10:30am UTC): This also affects the statistics of the group, though that's not a big concern.
- **h1_analyst_ava** (HackerOne triage) (June 10, 2019, 4:45pm UTC): Thank you for your submission. Your report is currently being reviewed and the HackerOne triage team will get back to you once there is additional information to share.
- **yashrs** (June 10, 2019, 5:02pm UTC): Were you able to reproduce the issue? Let me know if you need any more info/help regarding this.
- **h1_analyst_ava** (June 11, 2019, 2:32pm UTC): Apologies for the delay. We are still reviewing this report and will let you know as soon as we can provide an update.
- **h1_analyst_ava** changed the status to **Needs more info**. (June 11, 2019, 8:53pm UTC): After a few tries I have not successfully recreated your POC. Can you update this report to include a video and the BurpSuite configuration you used?
- **yashrs** changed the status to **New**. (June 12, 2019, 4:23am UTC): The video is already present in the second comment which I made. I used Turbo Intruder with https://github.com/PortSwigger/turbo-intruder/blob/266da04d9096dbcd4036e4549595d06d3e38a12a/resources/examples/race.py. Replace 30 concurrent threads with 5/10 to reproduce.
- **h1_analyst_ava** (June 13, 2019, 7:57pm UTC): Thank you for providing the requested updates. H1 is reviewing the additional information and will get back to you should there be any questions or comments.
- **h1_analyst_ava** (June 13, 2019, 11:50pm UTC): I am waiting for my invitation link to become active. If no one gets to this report before I do I'll pop back in to validate this one the link is available.
- **yashrs** (June 14, 2019, 12:09am UTC): You don't need to wait for it to become active. You can just copy it from source and it will should work on other account.
- **h1_analyst_ava** updated the severity from low to low (2.4). (June 14, 2019, 3:24pm UTC)
- **h1_analyst_ava** changed the status to **Triaged**. (June 14, 2019, 3:24pm UTC): We were able to validate your report, and have submitted it to the appropriate remediation team for review. They will let us know the final ruling on this report, and when/if a fix will be implemented. Please note that the status and severity are subject to change.
- **yashrs** (June 24, 2019, 4:38pm UTC): Do we have any updates for this report?
- **h1_analyst_ava** (June 24, 2019, 10:25pm UTC): It looks like the latest news on this is that it's been passed along to the proper parties for review.
- **yashrs** (July 10, 2019, 5:51am UTC): Just wanted to ask if we heard back from the team and whether they consider this a valid issue that they are going to fix or not.
- **jobert** (HackerOne staff) (July 10, 2019, 5:29pm UTC): Hi @yashrs - we haven't heard back from the team yet. We'll ping them again and get back to you when we know more. Appreciate the diligence!
- **HackerOne** rewarded yashrs with a bounty. (August 16, 2019, 4:16pm UTC): Thanks again for bringing this to our attention. The team is planning to get a fix out for this in the next three weeks.
- **yashrs** (August 16, 2019, 4:30pm UTC): Thanks a lot for the bounty :) This is a low priority/severity bug so I understand the delay
- **bencode** (HackerOne staff) (March 18, 2020, 2:35am UTC): Hey @yashrs, we have deployed a fix to prevent this from happening, can you confirm?
- **yashrs** (March 18, 2020, 4:43pm UTC): Hi @bencode, can confirm :)
- **bencode** closed the report and changed the status to **Resolved**. (March 18, 2020, 5:26pm UTC): Yes! Thanks! Sorry it took us so long, but glad it's resolved
- **jobert** requested to disclose this report. (March 20, 2020, 5pm UTC)
- **yashrs** (March 20, 2020, 5:08pm UTC): Hi @jobert, could you please remove/redact attachment [redacted by HackerOne]?
- **jobert** (March 20, 2020, 5:09pm UTC): Hi @yashrs - sure, I just removed it!
- yashrs agreed to disclose this report. (March 20, 2020, 5:17pm UTC)
- This report has been disclosed. (March 20, 2020, 5:17pm UTC)

Participants: yashrs, jobert, bencode, h1_analyst_ava.
