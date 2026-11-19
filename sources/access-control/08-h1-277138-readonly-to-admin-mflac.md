---
title: "Privilege Escalation: Read-Only to Admin"
author: foobar7 (reporter), disclosed on Inflection program
url: https://hackerone.com/reports/277138
date_published: 2019-03-15
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/277138
bounty: not stated (bounty awarded 2017-11-02 but amount hidden on the public page)
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- STATUS: PARTIAL. The public disclosed page exposes only the program's summary, metadata and activity timeline. The reporter's original report body (vulnerability_information) and all comment text are empty/not public (confirmed via the page scrape and the public report JSON endpoint). No bypass attempted. -->

# Report #277138: Privilege Escalation: Read-Only to Admin

Program: Inflection (https://hackerone.com/inflection)
Reporter: foobar7 (https://hackerone.com/foobar7), Clear-verified badge
State: Resolved
Severity: High (8.2)
Weakness: Privilege Escalation
CVE ID: None
Bounty / Retest / Total reward: Hidden
Reported on: October 14, 2017, 7:34am UTC
Disclosed: March 15, 2019, 5:10pm UTC
Participants: foobar7, jwilliams-inflection, khalidamin, mmuller-inflection
Duplicate: #281501 (filed by khalidamin)

## Summary by Inflection

While the interface hides the users page from read-only users, they can still perform PUT requests to the API to change their privileges where they only have read-only permissions.

## Timeline

- foobar7 submitted a report to Inflection. October 14, 2017, 7:34am UTC
- mmuller-inflection posted a comment. October 17, 2017, 8:25pm UTC
- foobar7 posted a comment. October 18, 2017, 5:57pm UTC
- khalidamin filed a duplicate (#281501) and was invited to participate in this report. October 23, 2017, 6:34pm UTC
- mmuller-inflection updated the severity from high to high (8.2). October 26, 2017, 7:05pm UTC
- mmuller-inflection changed the status to Triaged. October 26, 2017, 7:26pm UTC
- Inflection rewarded foobar7 with a bounty. November 2, 2017, 8:46pm UTC
- foobar7 posted a comment. November 4, 2017, 10:40am UTC
- mmuller-inflection closed the report and changed the status to Resolved. November 17, 2017, 11:08pm UTC
- khalidamin removed khalidamin as a participant. November 19, 2017, 4:43pm UTC
- foobar7 posted a comment. November 20, 2017, 4:55pm UTC
- foobar7 requested to disclose this report. March 7, 2019, 11:40am UTC
- jwilliams-inflection agreed to disclose this report. March 15, 2019, 5:10pm UTC
- This report has been disclosed. March 15, 2019, 5:10pm UTC

Account details: None. Scope tag: StripeM-Inner.
