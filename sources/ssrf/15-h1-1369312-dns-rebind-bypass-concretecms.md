---
title: "SSRF mitigation bypass using DNS Rebind attack"
author: adrian_t
url: https://hackerone.com/reports/1369312
date_published: 2022-11-25
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1369312
bounty: not stated
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: concretecms. Submitted: 2021-10-13. State: Resolved. Severity: {"rating": "low", "score": 3.5}. Weakness: Server-Side Request Forgery (SSRF).

## Report body (reporter)

We noticed that the upload functionality contains the ability to upload files from remote server, however there are some mitigations against accessing the AWS Instance Metadata service.

We've managed to bypass these mitigations using DNS rebinding and we've managed to fetch the AWS IAM keys when Concrete CMS is running in the cloud.

We've used http://1u.ms/ service for DNS rebinding, please see screenshots with evidence.

## Impact

An attacker can bypass the SSRF protections and he can fetch the AWS IAM keys under which the application is running. From here on he can do enumeration and mount other attacks.

