---
title: "SSRF in the Custom Integration Webhook discloses AWS metadata"
author: mohammad_obaid
url: https://hackerone.com/reports/643278
date_published: 2020-03-23
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/643278
bounty: $1,500
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: dynatrace. Submitted: 2019-07-15. State: Resolved. Severity: {"rating": "critical", "score": 10.0}. Weakness: Server-Side Request Forgery (SSRF).

## Summary (team, by pascalsec)

Dynatrace allows customers to set up a webhook integration in order to automatically send updates to a specific endpoint of choice. This submission identified a way how the setup made it possible to accidentally obtain AWS metadata. Dynatrace remediated the vulnerability and found no evidence of abuse associated with it. 

This is also mentionend in https://www.dynatrace.com/support/help/whats-new/release-notes/managed/sprint-186/.

## Timeline (event log, comment text not public)

- 2019-07-15 mohammad_obaid submitted report to Dynatrace
- 2019-07-16 triager antenna: severity set to critical (10.0), title changed, status Triaged, weakness changed
- 2019-07-16 Dynatrace rewarded $1,500 bounty
- 2019-07-17 pascalsec closed as Resolved, requested disclosure
- 2020-03-23 reporter agreed, report disclosed
- Dynatrace release notes referenced: https://www.dynatrace.com/support/help/whats-new/release-notes/managed/sprint-186/
