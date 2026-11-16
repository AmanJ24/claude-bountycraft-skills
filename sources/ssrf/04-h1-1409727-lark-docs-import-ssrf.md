---
title: "Full read SSRF via Lark Docs `import as docs` feature "
author: sirleeroyjenkins
url: https://hackerone.com/reports/1409727
date_published: 2022-01-28 (disclosed; reported 2021-11-24)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1409727
bounty: "$5,000"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: lark_technologies | State: Resolved | Severity: "critical" 9.6 | Weakness: Server-Side Request Forgery (SSRF)

## Report body (vulnerability_information)

(empty in public JSON)

## Summary by team

A SSRF (server side request forgery) vulnerability was found in the LarkDocs using the "import as docs" feature, which could have potentially been used to access services running on the internal network. We thank @sirleeroyjenkins for reporting this to our team and confirming the resolution.
