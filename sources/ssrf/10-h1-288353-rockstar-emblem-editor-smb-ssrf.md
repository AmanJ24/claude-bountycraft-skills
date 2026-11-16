---
title: "SMB SSRF in emblem editor exposes taketwo domain credentials, may lead to RCE"
author: alexbirsan
url: https://hackerone.com/reports/288353
date_published: 2018-01-12 (disclosed; reported 2017-11-08)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/288353
bounty: "$1,500"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: rockstargames | State: Resolved | Severity: "high"  | Weakness: Server-Side Request Forgery (SSRF)

## Report body (vulnerability_information)

(empty in public JSON)

## Summary by team

In this report, the researcher found that by submitting crafted SVG files, he was able to establish a listener on our server that enabled SSRF attacks. This potentially could have been pivoted to carry out more damaging attacks as well. We improved our validation of user-submitted SVG files to prevent this from happening in the future.

## Summary by researcher

I found that Imagemagick would happily send the users' login and NTLMv2 hash to remote attackers when processing UNC paths. This could be leveraged to crack the server password offline or potentially in an SMB relay attack.

Thanks to Dirk Lemstra for fixing this in Imagemagick too!
https://github.com/ImageMagick/librsvg/commit/f9d69eadd2b16b00d1a1f9f286122123f8e547dd
