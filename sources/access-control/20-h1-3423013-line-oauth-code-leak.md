---
title: "page.line.me Open Redirect Leading to OAuth Authorization Code Exposure and Access Token Compromise"
author: imnotr3al (reporter), program LY Corporation (LINE)
url: https://hackerone.com/reports/3423013
date_published: 2026-06-02
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/3423013
bounty: $1,000
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- STATUS: PARTIAL. The disclosed report page rendered only the program's summary, timeline and metadata. The reporter's full report body, PoC steps and individual comment texts were not included in the public scrape (no sign-in bypass attempted). Signed S3 avatar URLs / CSRF token stripped as [REDACTED]. -->

Report #3423013. Upvotes: 43.

**Title:** page.line.me Open Redirect Leading to OAuth Authorization Code Exposure and Access Token Compromise

**Summary by LY Corporation**

An open redirect vulnerability was identified in page.line.me because redirect destinations were not properly restricted to trusted domains. By abusing this behavior within an OAuth 2.0 authorization flow, an attacker could cause the authorization response to be sent to an attacker-controlled endpoint. This could expose the authorization code issued after successful user authentication, which may then be exchanged for a valid access token. As a result, the vulnerability could lead to unauthorized access to the victim's account, allowing an attacker to perform actions with the victim's privileges.

## Timeline

- imnotr3al (ID-verified hacker) submitted a report to LY Corporation. November 12, 2025, 12:30pm UTC
- imnotr3al posted comments: Nov 26, 2025 (1:49pm, 2:23pm), Nov 27, 2025 (4:55pm UTC)
- calvin55 (LY Corporation staff) changed the status to **Triaged**. December 2, 2025, 3:47am UTC
- shiki_ (LY staff) posted a comment. Dec 12, 2025; imnotr3al replied Dec 12, 2025
- 03sunf_ (LY staff) posted a comment. Jan 21, 2026; imnotr3al replied Jan 21, 2026
- 03sunf_ commented Jan 27, 2026
- imnotr3al requested mediation from HackerOne Support. Jan 27, 2026
- 03sunf_ commented Feb 2, 2026; imnotr3al replied Feb 2, 2026
- calvin55 commented Feb 3, 2026; imnotr3al replied Feb 5, 2026 and Apr 4, 2026
- calvin55 commented Apr 9, 2026 and Apr 14, 2026; imnotr3al replied Apr 14, 2026
- LY Corporation rewarded imnotr3al with a **$1,000** bounty. April 20, 2026, 9:03am UTC
- calvin55 closed the report and changed the status to **Resolved**. April 20, 2026
- imnotr3al requested to disclose this report. April 20, 2026
- calvin55 commented Apr 23, 2026; changed the report title Jun 2, 2026
- calvin55 agreed to disclose this report. June 2, 2026, 3:30am UTC
- This report has been disclosed. June 2, 2026, 3:30am UTC

## Metadata

- Reported on: November 12, 2025, 12:30pm UTC
- Reported by: imnotr3al
- Reported to: LY Corporation (https://hackerone.com/line)
- Participants: imnotr3al, calvin55, 03sunf_, shiki_, h1_analyst_minnie, h1_analyst_lakshya
- Report Id: #3423013
- State: Resolved
- Severity: High (7.1)
- Disclosed: June 2, 2026, 3:30am UTC
- Weakness: None
- CVE ID: None
- Bounty: $1,000
- Retest: None
- Total reward: None
