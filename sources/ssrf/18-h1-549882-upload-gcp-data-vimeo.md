---
title: "SSRF  leaking internal google cloud data through upload function [SSH Keys, etc..]"
author: dphoeniixx
url: https://hackerone.com/reports/549882
date_published: 2019-12-13
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/549882
bounty: not stated
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: vimeo. Submitted: 2019-04-28. State: Resolved. Severity: {"rating": "critical", "score": 9.0}. Weakness: Server-Side Request Forgery (SSRF).

## Summary (team, by tucvimeo)

Using our upload feature, the user was able to force an SSRF to occur.

## Summary (researcher, by dphoeniixx)

For more information you can read my writeup: https://medium.com/@dPhoeniixx/vimeo-upload-function-ssrf-7466d8630437

