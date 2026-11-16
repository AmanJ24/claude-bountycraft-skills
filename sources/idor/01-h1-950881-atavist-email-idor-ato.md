---
title: "IDOR when editing email leads to Account Takeover on Atavist"
author: bugra
url: https://hackerone.com/reports/950881
date_published: 2020-08-04
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/950881
bounty: not stated (awarded, amount hidden)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR when editing email leads to Account Takeover on Atavist

Reported to: Automattic | Reporter: bugra | Reported: 2020-08-04 | Disclosed: 2020-11-18 | State: Resolved | Severity: critical

## Report body

## Summary:
Hi team,
I created an account on Atavist and checked my settings page.
I can change my email at https://magazine.atavist.com/cms/reader/account with this request :

[attachment F936117 not captured]

And as you can see, there is a `id` parameter on request data. It's our user ID, and it's vulnerable for IDOR. So we can change any user's email address.

Also user IDs are sequential so an attacker can change all accounts' email.

## Steps To Reproduce:

  1.Go to https://magazine.atavist.com/login and Login to your account
  1. Go to https://magazine.atavist.com/cms/reader/account and open your proxy program 
  1. Change the email and click `Save`
  1. In request, change the ID to your test account's ID
  1. Forward the request
  1. Now you can reset victim's password via https://magazine.atavist.com/forgot

## Impact

Account Takeover without user interaction

Thanks,
Bugra

Image attached: F936117 request.PNG (request to change email; contains `id` parameter). Image content not captured.

## Timeline

- 2020-08-04 14:08 UTC: bugra submitted the report to Automattic.
- 2020-08-04 14:19 UTC, bugra: "Also, this endpoint accepts any email (even the registered emails), I think you should fix it too."
- 2020-08-05, xknown (Automattic staff): standard acknowledgement.
- 2020-08-12, bugra: pinged for update.
- 2020-09-08, xknown: "We deployed a patch to address this issue, can you please test if you can bypass it?"
- 2020-09-08, bugra: confirms the fix, could not find a bypass. Notes the email field still accepts already-registered emails, unsure if it is a security issue because user IDs differ.
- 2020-09-28, bugra: asks for the report to be closed. 2020-10-30: bugra requested mediation from HackerOne Support. 2020-11-05: smilingquokka (H1 support) says the team is looking into it.
- 2020-11-05 10:28 UTC: xknown closed the report as Resolved. Automattic rewarded a bounty (amount hidden). Comment edited to note a related comment added in #951292.
- 2020-11-05, bugra: argues this is critical and that the bounty equals what was paid for a medium reflected XSS (#947790); says he was paid $650-$750 on earlier CrowdSignal account takeover reports; asks to reconsider.
- 2020-11-05, xknown: cannot give specifics on bounty amounts, "we don't award the same amount for every affected domain."
- 2020-11-18: bugra requested disclosure; xknown agreed; report disclosed 2020-11-18 14:21 UTC.

## Metadata (from report sidebar)

Reported to: Automattic. Severity: Critical (9 ~ 10). Weakness: Insecure Direct Object Reference (IDOR). State: Resolved. CVE: none. Bounty: hidden on page (report notes a bounty was awarded).
