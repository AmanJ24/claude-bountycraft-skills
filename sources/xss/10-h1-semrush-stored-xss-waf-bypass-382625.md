---
title: Stored XSS in '' Section and WAF Bypass (Semrush, Report #382625)
author: jimgogogo
url: https://hackerone.com/reports/382625
date_published: 2018-07-17 (disclosed 2018-12-07)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/382625
bounty: awarded plus WAF-bypass bonus, amount hidden
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Reported to: Semrush. Reported by: jimgogogo. Reported on: July 17, 2018, 2:32pm UTC. Status: Resolved. Severity: Medium (6.5). Weakness: Cross-site Scripting (XSS) - Stored. Disclosed: December 7, 2018.

## Summary

Stored Cross-site Scripting (XSS) is the most dangerous type of Cross Site Scripting. Web applications that allow users to store data are potentially exposed to this type of attack. stored XSS occurs when a web application gathers input from a user which might be malicious, and then stores that input in a data store for later use.

## Description

I have found this Stored XSS in 'position tracking' of SEMrush website.
first I setup 'position tracking' then in 'Rankings Distribution' tab add a valid domain in competitor's domain field which is check the validation just in client side so I try to hook it via BurpSuit and change the domain parameter to my XSS payload, as I see it saved my payload completely but by the time it wants to show me, my payload must have passed through the website Firewall so there is nothing to show.
As I guess the Firewall probably uses blacklist that controls just the HTTP/GET method so Although it blocked lots of HTML tags and attributes but I try hard and bypass it after all as shown in Figures Blocked_by_WAF.png (F321488) and Failed_1.jpg (F321489).
in conclusion, there is a WAF bypass vulnerability besides the XSS that may cause of effects in other vulnerabilities happening.

## Browsers Verified In

Mozilla 5.0 Firefox 52.0 ESR

## Steps To Reproduce

1. Log in to your account.

2. Create project.

3. Navigate to Dashboard-> Position Tracking ->Rankings Distribution -> select one of add domains -> click on "Edit competitor's list"

4. Fill 'new competitor's domain' field with a valid domain like google.com then open BurpSuit for changing the domain parameter with the payload below then click on "Add to list"

`"><u>XSS Vulnerability</u><marquee+onstart='alert(document.cookie)'>XSS`

    Note: lots of tags and attributes are blocked by the firewall but this rare payload isn't blocked so far and works correctly as shown in Figure WAF_Bypassed.png (F321486).

5. After changing the domain parameter click on 'Update' button then close "Position Tracking Settings" page.

6. And shown in Figure Resualt.jpg (F321490) after closing the "Position Tracking Settings" page the XSS is loaded.

## Supporting Material/References

Images: F321488 Blocked_by_WAF.png, F321489 Failed_1.jpg, F321486 WAF_Bypassed.png, F321490 Resualt.jpg (not captured). Reference: https://www.owasp.org/index.php/Testing_for_Stored_Cross_site_scripting_(OTG-INPVAL-002)

## Impact

The attacker can have the session and cookie of customers and deface that page.
The firewall that uses blacklist is bypassed by the special payload.

## Timeline

- Jul 18, 2018 (slowstock, Semrush staff): "Thanks for the report, we will investigate this." Later: "Could not reproduce. Could you send the video, please."
- Jul 18, 2018 jimgogogo: "I beg you a pardon for delay. It's because I'm doing WAF bypass as well so it's maybe a bit longer." Attached video SEMrush_XSS_Bypass.mp4 (F321885).
- Jul 19, 2018: severity updated from medium (6.5) to low (3.5). Jul 20: Triaged. Jul 26: xss03111 filed duplicate #386791 and was invited. Jul 27: severity updated back from low (3.5) to medium (6.5).
- Jul 27, 2018: Semrush rewarded jimgogogo with a bounty: "Thanks for making our service safer! Also we decided to set a bonus for WAF bypass."
- Aug 13, 2018 (sergin): "Could you please check the fix?" jimgogogo: "Although the vulnerability is resolved, But the WAF is still bypassing with my payload."
- Aug 14, 2018 xss03111: "To bypass WAF payload should be in new line (For Example)"

```
firstName=ahmed

<script>alert(0);<script>&lastName=mohamed
```

  "if it was : firstName=ahmed<script>alert(0);<script>&lastName=mohamed  WAF won't bypassed"
- Aug 27/29, 2018: Semrush asked again to check fix; jimgogogo repeated that the WAF was still bypassed with his payload (images F339321 WAF_bypass.png, F339322 WAF_ok.png).
- Sep 5, 2018 (slowstock): "We're currently working on adding new rules to our WAF, and now we're closing the report because the vulnerability is resolved. Thank you for making our service safer!" Status Resolved.
- Nov 16, 2018 disclosure requested; Dec 7, 2018 disclosed by Semrush.

## Metadata

Report Id #382625. Participants: jimgogogo, security_semrush, slowstock, averonesis, sergin, xss03111. CVE ID: None. Bounty: Hidden.
