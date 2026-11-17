---
title: "Incorrect logic when buy one more license which may lead to extend the expire date of existing license"
author: liru
url: https://hackerone.com/reports/2461737
date_published: 2024-04-13
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/2461737
bounty: not stated
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #2461737 to PortSwigger Web Security. Status: Informative. Severity: No rating. Weakness: Business Logic Errors. Disclosed April 16, 2024. Bounty: hidden/not shown. Some details redacted by the program in the public version.

## Report body (liru, April 13, 2024)

Hi Team,

I noticed a bug in the licenses which may lead to extend the expire date of existing license. To be honest, it is hard for me to reproduce it. I was plan to see if the license still works after [REDACTED]. I think it's better to report this issue to you although it may prove it is just a display issue.

Background: when our company buy one more license with 5 years. Our existing license with 4 users will expire on [REDACTED]. After we pay the money and got the new license. I revisit the following page and found something interesting. My existing license change from 4 users to 1 user, but the expire date is change to [REDACTED].

1. Make sure you have an existing license (in my case, we have a 4 users licenses which will expire on [REDACTED]).
2. Buy a new one user license with 5 years and pay it.
3. After you receive the license, check it on the following page. [REDACTED]

Observed result:

1. The existing license has change from 4 user to 1 user, but the expire date is [REDACTED]. I don't try to see if we can still use the left of the license because it will consume our licenses.
2. The new license should expire at around [REDACTED] but now it is [REDACTED]. This means the new license expire date is extended.

You can track my account to debug this issue. If you need any help from me. Please feel free to let me know. Thanks!

User can use this way to extend the existing license. For example, you can just buy one year and it will expire next year. Then buy one more license which will expire 10 years. Now you have two license both will expire 10 years.

(Follow-up comments: a video showing the page was attached; redacted.)

## Timeline / vendor responses

- April 15, 2024, albinowax (PortSwigger staff): Thanks for the report, I'll check this with the team to see if there's an issue here.
- April 16, 2024, albinowax closed as Informative: "It looks like you've hit a bug here, but it's left your account in a worse state: The payment notification from your payment was processed twice. Your account was downgraded to 2 users when it should have 4. We will manually issue you with a temporary 3 user license that will ensure you have the number of users you paid for until the renewal date, and fix the bug that caused the downgrade. Please note that since the bug left your account in a worse state, we don't consider it to be a vulnerability."
- liru disagreed: one of the orders did extend the expire date; asks what happens if the existing license is only one user + one year (upgrade or downgrade?). Requested disclosure.
- albinowax agreed to disclose. Note: "the bug that caused the downgrade has not been fixed yet. If you or anyone reading this is able to provide working replication steps that leave your account at an advantage, we'd love to hear about it."
- liru later proposed proving it on their own account (buy 1 user + 1 year, then 1 user + 2 years, expecting both to expire in 2 years) with refund afterward.

## Metadata
Reported on April 13, 2024. Reported to PortSwigger Web Security. Participants: liru, albinowax. CVE: none. Account details: none.
