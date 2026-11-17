---
title: "Stored XSS on activity"
author: shazadsadiq
url: https://hackerone.com/reports/391390
date_published: 2018-08-14
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/391390
bounty: awarded (amount hidden/not stated)
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# Stored XSS on activity

## Report metadata (HackerOne public JSON)

- Program: shopify
- Reporter: shazadsadiq
- Submitted: 2018-08-07T17:31:29.764Z
- Disclosed: 2018-08-14T20:29:30.810Z
- State: Closed / Resolved
- Weakness: Cross-site Scripting (XSS) - Stored
- Severity: {"rating": "high", "author_type": "User"}

## Report body

Hi security team members,

#Description
I found a store xss on the activity which allows an attacker to steal admin account cookies.

#Step to reproduce
1-Create store
2- Add a member in a store
3- Member can choose any name 
4- So change the any member name with hunter"><svg/onload=alert(2)>
5- Now on admain account make changes 
6- That will create activity with attacker malicious payload

#POC
Please see the below image
{F329469}
Let me know if more information is needed to my end.
Best Regards,
Shahzad

## Impact

An attacker(staff member) can takeover admin account.

## Attachments (names only)

- Untitled.png (image/png)
