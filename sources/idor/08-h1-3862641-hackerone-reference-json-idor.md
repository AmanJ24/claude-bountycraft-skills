---
title: "IDOR allows user to access report details via reference.json endpoint"
author: sm41ldrag0n79_mbbank
url: https://hackerone.com/reports/3862641
date_published: 2026-07-14
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/3862641
bounty: $15,000
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR allows user to access report details via reference.json endpoint

Reported to: HackerOne | Reporter: sm41ldrag0n79_mbbank | Reported: 2026-07-14 | Disclosed: 2026-10-01 | State: Resolved | Severity: critical

## HackerOne summary (public)

**Summary:**
An information disclosure vulnerability on the `/reports/{id}/reference.json` endpoint allowed VDP accounts created via `hackerone.com/vdp-sign-up` to access certain private report details. While the endpoint returned HTTP 422 "You do not have the appropriate access," the error response contained additional report details in the `context.subject` field.

**Description:**
By sending a POST request to the `/reports/{report_id}/reference.json` endpoint with an empty JSON body `{}`, VDP users could retrieve information about private reports belonging to other programs despite receiving an access-denied error. Although the endpoint correctly rejected unauthorized access with an HTTP 422 status code, it inadvertently included private report data in the error response body.

**Steps To Reproduce:**

1. Create a VDP account at `hackerone.com/vdp-sign-up`
2. Log in with the VDP account
3. Obtain an approved test report ID from https://hackerone.com/security?type=team (these reports contain flags to include in your POC), 
4. Send a POST request to `https://hackerone.com/reports/[TEST_REPORT_ID]/reference.json` with the following structure:

```
POST /reports/[TEST_REPORT_ID]/reference.json HTTP/2
Host: hackerone.com
Cookie: [your session cookie]
Content-Type: application/json
X-Csrf-Token: [your CSRF token]
Content-Length: 2

{}
```

4. Observe that despite receiving HTTP 422 with "You do not have the appropriate access," the response contains report details in the `context.subject` field

**Note:** This vulnerability has been fixed. We have found no evidence of misuse in our investigation to date, and no action is required from users..

Note: the body of the original report was not exposed in the public JSON (`vulnerability_information` empty). The public page shows only the HackerOne "Summary" above. Most timeline comments are collapsed and their text is not shown on the public page; only the activity skeleton is visible. A "Submitted with Report Assistant" tag appears at the top of the page.

## Timeline (activity skeleton visible on public page)

- 2026-07-14 10:18 UTC: sm41ldrag0n79_mbbank submitted the report to HackerOne.
- 2026-07-15: h1_mediation_neal (HackerOne triage) posted a comment. HackerOne rewarded a $15,000 bounty (2026-07-15 19:37 UTC).
- 2026-07-15 to 2026-09-26: several comments by the reporter (text not shown). jerboa (HackerOne staff) changed the report title (2026-07-17, 2026-10-01) and commented.
- 2026-07-21: z0x01 (HackerOne staff) changed status to Retesting; reporter completed a retest; HackerOne accepted the retest; z0x01 closed the report as Resolved.
- 2026-09-21: reporter requested disclosure. 2026-10-01: jerboa cancelled and re-requested disclosure; reporter agreed; report disclosed 2026-10-01 01:27 UTC.

## Metadata (from report sidebar)

Reported to: HackerOne (managed). Severity: Critical (9 ~ 10). Weakness: IDOR. State: Resolved. Bounty: $15,000. CVE: none.
