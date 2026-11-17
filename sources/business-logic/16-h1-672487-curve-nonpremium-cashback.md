---
title: "Business Logic Flaw - A non premium user can change/update retailers to get cashback on all the retailers associated with Curve"
author: praseudo7
url: https://hackerone.com/reports/672487
date_published: 2019-08-13
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/672487
bounty: not stated (bounty plus extra "+bonus" awarded, amounts hidden)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

#672487 Business Logic Flaw - A non premium user can change/update retailers to get cashback on all the retailers associated with Curve

Status: Resolved. Severity: Medium (4.3), downgraded from high. Weakness: Business Logic Errors. Reported Aug 13, 2019 15:07 UTC. Disclosed Jul 24, 2020.

Hi,

While testing your android application I've found a business logic flaw by using which a non premium user can update/change the retailers when ever and what ever retailers he wants to.

Curve application has a functionality called "Earn curve cash". A non premium user can select only 3 retailers (where as premium user can select 6 or more retailers) at a time. A business logic flaw exists at this endpoint by using which a non premium user can update/change already existing retailers and can use cashback with all the retailers associated with curve application.

# Steps to reproduce:

1] Login to non premium user account
2] Navigate to "Earn curve cash" and you can select only 3 retailers (because you are non premium user)
3] And click on "Confirm"
4] You have now added 3 retailers to your account and you dont have any option to edit/update the added retailers.
5] Now go back to "Earn curve cash" functionality and make sure to capture the request and response

# Request:

```
GET /v1/rewards/users/programs/<program-uuid>/merchants HTTP/1.1
Accept: application/json
Curve-AppAndVersion: Curve Android 2.9.0
crv-user-agent: Android 2.9.0/20900
Authorization: [REDACTED]
crv-idempotency-key: <uuid>
crv-request-id: <uuid>
crv-correlation-id: android-<uuid>
Host: api.imaginecurve.com
Connection: close
User-Agent: okhttp/3.12.2
```

# Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
server: envoy

{"success":true,"data":{"merchants":[{"id":"<uuid>","name":"Waitrose","alias":"waitrose","hidden":false,"countries":["GBR"],"category":{"id":"<uuid>","name":"Groceries","curve_category_id":10},"percentage":1},{"id":"<uuid>","name":"Whole Foods","alias":"wholefoods", ... },{"id":"<uuid>","name":"Tesco","alias":"tesco", ... }]}}
```

6] Now change the response body with below JSON content and turn off the burp interception

```
{"success":true,"data":{"merchants":[]}}
```

7] Now you can again select any three new retailers and click on "Confirm"
8] You have now successfully updated/changed the retailers and can use the cashback on all the retailers associated with curve application.

For better understanding I've attached video POC (curve_business_logic.mp4, not captured).

## Impact

A non premium user can miss-use the business logic and can use cashback's with all the merchants associated with the curve application.

# Mitigation:

Make sure to validate the responses and user based authorization for each endpoint on the server-side.

Regards,
Praseudo

## Timeline / comments

- Aug 15, 2019: ktistai (H1 triage) standard acknowledgement.
- Aug 17: praseudo7 asks for update.
- Aug 19: lasagna (H1 triage) set Needs more info: "We're unable to reproduce your finding. When I click on the 'See Retailers' button in 'Earn curve cash' I'm getting redirected to an external website instead. I don't see any request going to /v1/rewards/users/programs/[id]/merchants. Can you recheck your reproduction steps?"
- Aug 19: praseudo7: Once retailers are added to a premium account you cannot add or update them, that is the intended functionality, but here a user can bypass it. Reproduced the same steps, video PoC attached, asks for triager's video.
- Aug 21: praseudo7 asks again.
- Aug 27: lasagna updated severity from high to medium (4.3), changed status to Triaged: validated and submitted to remediation team.
- Sep 2: praseudo7 asks for update; lasagna says no feedback from the team yet.
- Sep 8, 2019: olouhaidia closed as Resolved; Curve rewarded a bounty: "thanks @praseudo7 - great report."
- Sep 9-10: praseudo7 notes the program page promised an extra $500 bonus for valid Critical, High and Medium reports, asks why not added (screenshot of program page attached).
- Oct 10, 2019: Curve rewarded another bounty: "+bonus! sorry it took that long".
- Oct 14, 2019: olouhaidia reopened and re-closed as Resolved.
- Jun 24, 2020: praseudo7 requested disclosure. Jul 24, 2020: disclosed.

Participants: praseudo7, olouhaidia, lasagna, ktistai. Bounty, retest, total reward: Hidden.
