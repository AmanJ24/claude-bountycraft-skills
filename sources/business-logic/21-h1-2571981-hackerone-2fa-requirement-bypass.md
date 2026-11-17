---
title: "Business Logic error leads to bypass 2FA requirement"
author: abdulprkr
url: https://hackerone.com/reports/2571981
date_published: 2024-06-24
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/2571981
bounty: not stated
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #2571981 to HackerOne. Status: Duplicate of #694805. Severity (as rated on report): High (8.2). Weakness: Business Logic Errors. Bounty: none. Disclosed July 11, 2024 by HackerOne staff (unilateral disclosure as part of an MFA spot check).

## Report body (abdulprkr, June 24, 2024)

Hi team,

### Summary
I have identified a business logic issue in the 2FA requirement. I noticed that the organization enables the 2FA requirement so that only reporters who have set up 2FA can report, due to security reasons. This is because the report contains sensitive information, and if a hacker's credentials are compromised, the 2FA protection should be in place. This ensures that the vulnerability reported by the hacker remains secure. However, if the hacker adds another hacker as a collaborator, HackerOne does not check whether the invited hacker has set up 2FA or not. The invited hacker can join the report without any 2FA requirement, which contains the same sensitive information that the organization has mandated 2FA to protect. Therefore, it is necessary to ensure that the invited hacker also has 2FA set up. Otherwise, they should not be able to accept the invitation until they set up 2FA. This would ensure that only those hackers who have set up 2FA can access the organization's report.

### Steps to reproduce
1. Create 2 accounts, one with 2FA enabled (A) and another without 2FA (B)
2. Select a program which requires 2FA and allows collaboration
3. Create a report using account A and add account B as collaborator
4. Submit the report
5. Observe that the invitation is sent successfully
6. Accept the invitation and observe that you can access the report without the 2FA requirement

### Impact
- Sensitive information exposure: the purpose of 2FA is to secure sensitive information in reports. A collaborator without 2FA can access it without the additional layer.
- Increased risk of data breaches: if a hacker's credentials are compromised, 2FA is supposed to mitigate this. A collaborator without 2FA exposes the organization to potential breaches.

## Timeline / vendor responses
- June 29, 2024, HackerOne triage (h1_analyst_pablo) closed as Duplicate of #694805. Excerpt of the original report quoted by triage: title "2FA disabled External user can participate in 2FA enabled program's report"; triage says "The 2FA requirement is for people submitted reports to the program, not for being invited to a report."
- July 11, 2024: HackerOne staff disclosed the report. Disclosure reason: HackerOne was running a spot check on its MFA functionality to identify bypasses, and disclosed prior (mostly duplicate or informational) MFA reports for transparency and to avoid duplicate reports.

## Metadata
Reported on June 24, 2024. Reported to HackerOne (managed). Participants: abdulprkr, 0daybookworm, h1_analyst_pablo. CVE: none.

Key pattern: a security control (2FA requirement) is enforced on the primary actor (submitter) but not on a secondary path to the same data (collaborator invite). Vendor treated this as intended scope of the control.
