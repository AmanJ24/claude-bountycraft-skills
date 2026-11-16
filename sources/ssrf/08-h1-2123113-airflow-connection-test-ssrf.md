---
title: "SSRF Vulnerability through Connection test feature"
author: sayoojbkumar
url: https://hackerone.com/reports/2123113
date_published: 2023-09-12 (disclosed; reported 2023-08-25)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/2123113
bounty: "$2,550"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Program: ibb | State: Resolved | Severity: "medium"  | Weakness: Server-Side Request Forgery (SSRF)

## Report body (vulnerability_information)

An SSRF (Server-Side Request Forgery) attack can potentially be executed through the connection test feature, specifically when interacting with the Slack API. This attack allows an attacker to see the plain response, providing them with information that can aid in exploiting the vulnerability, rather than relying solely on blind SSRF techniques.

## Impact

This vulnerability allows the user to access connection information and exploit the test connection feature by sending many requests, leading to a denial of service (DoS) condition on the server. Furthermore, malicious actors can leverage this vulnerability to establish harmful connections with the server, hit on metadata endpoints.

{F2635041}

## Summary by team

Affected versions:

- Apache Airflow before 2.7.0

Description:

Apache Airflow, in versions prior to 2.7.0, contains a security vulnerability that can be exploited by an authenticated user possessing Connection edit privileges. This vulnerability allows the user to access connection information and exploit the test connection feature by sending many requests, leading to a denial of service (DoS) condition on the server. Furthermore, malicious actors can leverage this vulnerability to establish harmful connections with the server.

Users of Apache Airflow are strongly advised to upgrade to version 2.7.0 or newer to mitigate the risk associated with this vulnerability. Additionally, administrators are encouraged to review and adjust user permissions to restrict access to sensitive functionalities, reducing the attack surface.
