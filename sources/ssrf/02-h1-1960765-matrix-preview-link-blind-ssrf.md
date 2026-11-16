---
title: "Blind SSRF to internal services in matrix preview_link API"
author: la_revoltage
url: https://hackerone.com/reports/1960765
date_published: 2023-04-26 (disclosed; reported 2023-04-24)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1960765
bounty: "$6,000"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: reddit | State: Resolved | Severity: "high" 7.5 | Weakness: Server-Side Request Forgery (SSRF)

## Report body (vulnerability_information)

## Summary:
Reddit' new chat is based on Matrix software which has preview_link functionality which doesn't filter the URL before sending the request

## Impact:
Attacker can enumerate services by grabbing og:title and port scanning, also possible RCE escalation (Asking for permission on this one)

## Steps To Reproduce:


  1. Visit the https://matrix.redditspace.com/_matrix/media/r0/preview_url/?url=*
  2. Replace * with http://██████ to get og:title ███████
  3. Replace * with http://█████████ to get og:title ███████
 4. Replace * with http://██████████to get og:title ██████
 5. Replace * with ████████ to get og:title █████████

Note: If the request is stuck and not responding in 2 seconds reload the page until it does

## Permit for escalation attempt? 
Since the ███ URL is accessible it may be possible to run ███:
GET █████████

There are also possibilities to test ██████, but I thought that it would be incorrect to do such activity without permission and as such report vulnerability in this state. I also therefore request a permission to try to escalate this to Critical

## Impact

Attacker can enumerate services and launch attacks against them

## Summary by team

Matrix Chat endpoint at https://matrix.redditspace.com/_matrix/media/r0/preview_url/?url=* allowed partially blind SSRF to internal services. The data that could be exfiltrated was limited only to the service names and their IPs before a fix was implemented. This endpoint should not be able to query internal services, but external IPs, domains and services are fine for this to query.

## Summary by researcher

Matrix endpoint at https://matrix.redditspace.com/_matrix/media/r0/preview_url/?url= allowed Partially Blind SSRF which allows attacker to send GET requests and exfiltrate data about internal services
