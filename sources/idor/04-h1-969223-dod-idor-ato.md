---
title: "IDOR to Account Takeover on https://████/index.html"
author: nagli
url: https://hackerone.com/reports/969223
date_published: 2020-08-27
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/969223
bounty: none (VDP)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR to Account Takeover on https://████/index.html

Reported to: U.S. Dept Of Defense | Reporter: nagli | Reported: 2020-08-27 | Disclosed: 2020-09-29 | State: Resolved | Severity: critical

## Report body

Hello Team!

**Summary:**

I found when you wish to update your profile on https://███████/ after your login through https://██████████/signIn/signIn.html website due to an IDOR.

This IDOR gives you the opportunity to change the origin email for the registered account by changing the ID parameter on the following request, i assume that if i would do it on the id=1 i would takeover the admin account, this is due to not requiring the OLD password to make an email change, aswell as no restriction to make POST actions on different account IDS.


**Description:**

IDOR chained to full Account Takeover on ██████ domain.

Account
## Step-by-step Reproduction Instructions

1. Register an account at https://█████████/signIn/CreateAccount.html (Attacker)
2. Login to your account and go the https://███████/signIn/account page
3. Click on the "update" button located at thetop middle, and capture the request on BURP
4. Now change the ID parameter on the request to the victims, change the email, and you successfully have managed to switch his email.

Request:

███


Video PoC:

█████

## Suggested Mitigation/Remediation Actions

1. Implementing email request change based on OLD password input
2. Returning 403/401 when user account attempts to change another user ID settings.

## Disclaimer

as you might notice the domain is https://██████/signIn/signIn.html when you sign in (.mil site), and it redirects you afterwards to https://██████████/ with the Compromised account, therefore i considered this In Scope.

Best Regards,
Nagli

## Impact

Issuing the malicious request on the victim account ID will lead to account takeover by replacing the email of the victim with the email of the attacker, and requesting a new password using the Forgot password option.

Note: much of the original report (domain names, request, video) is redacted by the program on the public page. Title as shown: "IDOR to Account Takeover on https://████/index.html".

## Program summary shown at top of the report ("Summary by U.S. Dept Of Defense")

More details about the report could be found at my blogpost: https://naglinagli.github.io/DoD_IDOR/
I would like to thank the DoD for a quick triage and resolution regarding the issue.

## Timeline

- 2020-08-27 19:24 UTC: nagli submitted the report to U.S. Dept Of Defense.
- Automated DoD VDP greeting bot comment.
- 2020-08-27 20:21: agent-l8 (DoD VDP) set severity to high and status to Triaged ("We have validated the vulnerability you reported").
- 2020-09-16: nagli asks if resolved since the host redirects to Google and the sign-in page returns 404. agent-l8: "They are actively working it."
- 2020-09-25: agent-j5 (DoD staff) closed the report as Resolved.
- 2020-09-25: nagli requested disclosure. 2020-09-29: ag3nt-j1 (DoD staff) agreed; report disclosed and locked.
- 2020-11-17: ag3nt-j1: "I think it's fair and will go ahead and raise this to a critical finding." Severity changed from high to critical.

## Metadata (from report sidebar)

Reported to: U.S. Dept Of Defense (VDP). Severity: Critical (9 ~ 10). Weakness: IDOR. State: Resolved. Bounty: hidden/none (VDP). CVE: none.
