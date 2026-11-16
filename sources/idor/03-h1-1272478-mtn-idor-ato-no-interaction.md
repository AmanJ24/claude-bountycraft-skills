---
title: "IDOR Leads To Account Takeover Without User Interaction"
author: theranger
url: https://hackerone.com/reports/1272478
date_published: 2021-07-21
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1272478
bounty: none
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR Leads To Account Takeover Without User Interaction

Reported to: MTN Group | Reporter: theranger | Reported: 2021-07-21 | Disclosed: 2022-09-04 | State: Resolved | Severity: critical

## Report body

## Summary:
Hello Team,
There's IDOR Bug on this subdomain `mtnmobad.mtnbusiness.com.ng` leads to account takeover, More details check the Poc. 

## Steps To Reproduce:

  1. Create two accounts on `mtnmobad.mtnbusiness.com.ng` and both accounts verify the emails from your email inbox
  2. Login to attacker account on Browser A Go to update Profile Try to update the address for example and Capture the Request with burp send it to `Repeater`
[attachment F1384484 not captured]
3. Login to Victim account on browser B do the same to get the victim `ID` you can Grab his ID without sending this request to `Repeater`
4. Go to the Attacker Request You sent to `Repeater` Change `/ID` with the Victim's `ID` you Grabbed From Step 3 Then change `Email` with different email, you need to change the `username` parameter not the `email` see this screenshot, Leave the email as your attacker email. the `username` Value is email and just update that one.

[attachment F1384509 not captured] 
5.  Go Reset the Password (act like you don't know the Pass XD), login and successfully account Takeover without User Interaction

## Supporting Material/References:
--Check this Video :
[attachment F1384553 not captured]

## Impact

Full account Takeover without user interaction

Images attached (not captured): F1384484 Screenshot_1.png (update-profile request in Burp), F1384509 Screenshot_1.png (request modified with victim ID and `username` param). Video: F1384553 idor.mp4.

## Impact (as written)

Full account Takeover without user interaction

## Timeline

- 2021-07-21 18:40 UTC: theranger submitted the report to MTN Group.
- 2021-07-25, theranger update: numeric user IDs are weak, e.g. his two test accounts were `572` and `573`, so users on `571`, `570` and so on exist. "if an attacker exploited this with +500 emails he'll change every account Email Addresses on the system and reset the passwords for all the accounts on that specific domain."
- 2021-08-31: kdrmtn (MTN Group staff) changed status to Triaged: "I have logged a ticket with the responsible party to investigate."
- 2022-08-02: kdrmtn closed the report as Resolved.
- 2022-08-05: theranger requested disclosure. Report disclosed 2022-09-04.

## Metadata (from report sidebar)

Reported to: MTN Group (managed). Severity: Critical (9 ~ 10). Weakness: IDOR. State: Resolved. Bounty: None. CVE: none.
