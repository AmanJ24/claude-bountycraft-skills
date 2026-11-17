---
title: "Race Condition when following a user"
author: bugra (Bugra Eskici)
url: https://hackerone.com/reports/927384
date_published: 2020-07-19
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/927384
bounty: not stated (amount hidden on report)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Scrape status: FULL (complete report body and timeline rendered; bounty amount hidden by HackerOne). Presigned attachment URLs replaced by [REDACTED]. The Turbo Intruder script was garbled by the HTML-to-markdown render (whitespace lost); reconstructed below with the same tokens.

Report #927384: Race Condition when following a user
Reported to: Staging.every.org. Reported by: bugra. Reported on: July 19, 2020, 3:48pm UTC. Disclosed: September 9, 2020, 5:51am UTC.
Status: Resolved. Severity: Low (0.1 ~ 3.9). Weakness: Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition'). CVE: None. Bounty / Total reward: Hidden.

## Report body (bugra, July 19, 2020, 3:48pm UTC)

## Summary:

Hi team,
There is a race condition vulnerability when following a user. If you send the `Follow` requests asynchronously, you can follow a user multiple times instead getting an error message.
I've been using Turbo Intruder extension at Burp Suite for trying Race Condition attacks. I can recommend it for reproduce this vulnerability.

## Steps To Reproduce:

1. Go to any user's profile

2. Turn on Intercept at Burp Suite and click `Follow` button

3. Right click to follow request, click `Send to turbo intruder` and drop the request

4. Add a fake header that contains `%s` value. Like `Test: %s`

5. Paste this Python code to Turbo Intruder :

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=100,
                           pipeline=False
    )

    for i in range(30):
        engine.queue(target.req, str(i), gate='race1')

    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

6. Click `Attack` button. Turbo Intruder will send 30 requests, check the status codes. If you see multiple responses with `201 Created` status, that means you followed the user multiple times.

## PoC Video :

Video F913171: EveryRaceCondition.mp4 (4.59 MiB) [REDACTED presigned URL]

## Impact

Race Condition vulnerability allows to following a user multiple times with one account

Thanks,
Bugra

## Timeline

- **mu_yp51u** changed the status to **Triaged**. (July 19, 2020, 6:41pm UTC): Reported to engineering team, thank you for helping keep nonprofits more secure!
- **bugra** (July 19, 2020, 7pm UTC): Perfect! Let me know if you need more information or anything to help :)
- **bugra** (September 7, 2020, 12:34pm UTC): Hi @mu_yp51u, I checked the bug and it looks fixed. Is there any update from engineering team?
- **osdiab** closed the report and changed the status to **Resolved**. (September 7, 2020, 1:58pm UTC): Indeed, we made it idempotent so it should be fine now. Thank you for the report and for checking that it's fixed - these reports go a long way to protecting the privacy of donors and nonprofits!
- bugra requested to disclose this report. (September 7, 2020, 2:59pm UTC): @osdiab Perfect! Thanks for your quick response. Can we disclose this report?
- osdiab agreed to disclose this report. (September 9, 2020, 5:51am UTC)
- This report has been disclosed. (September 9, 2020, 5:51am UTC)

Participants: bugra, osdiab, mu_yp51u.
