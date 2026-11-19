---
title: "Account TakeOver Due to Improper Handling of JWT Tokens in usememos/memos"
author: M Nadeem Qazi (@mnqazi)
url: https://huntr.com/bounties/4747a485-77c3-4bb5-aab0-21253ef303ca
date_published: 2023-04-20
date_scraped: 2026-11-17
platform: other
disclosed_report: none
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- Scrape note: full public report text captured. The PoC is an external Google Drive video (not captured). Date is the "Reported on" date. huntr bounty page; no bounty amount shown. -->

# Account TakeOver Due to Improper Handling of JWT Tokens in usememos/memos (https://github.com/usememos/memos)

Status: Valid. Reported on Apr 20th 2023.

# Description

I have discovered a vulnerability where any user can modify another user's data including password simply by intercepting and changing the access token of the JWT using https://token.dev. The system does not verify whether the JWT token was issued by the server or not, allowing it to accept the edited JWT token. This can lead to unauthorized modification or changing of the email address, and password and other personal data of any user.

# Proof of Concept

External video PoC (Google Drive link, not captured).

# Impact

This vulnerability can have significant consequences on the security of the affected system. By allowing unauthorized users to modify the data of other users, it can result in the following:

Account Takeover: An Attacker can change the password of any user just by using forged JWT token.

Privacy violations: Any unauthorized modifications to a user's email can lead to privacy violations as sensitive information can be exposed or accessed without the user's consent.

Data breaches: If a user's email is changed, it can result in unauthorized access to sensitive information that may result in a data breach. This can lead to legal and financial consequences for both the users and the organization.

Attacker can use the further information to defame the users or organization.

## References

- https://connect2id.com/products/nimbus-jose-jwt/vulnerabilities

## Metadata

- CVE: CVE-2023-4696 (published)
- Vulnerability Type: CWE-284: Improper Access Control
- Severity: Critical (9.8)
- CVSS: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H (Network, Low complexity, no privileges, no user interaction, Unchanged scope, C/I/A High)
- Registry: Golang
- Affected Version: memos v0.12.2
- Visibility: Public
- Status: Fixed
- Found by: M Nadeem Qazi (@mnqazi)
- Fixed by: correctroadh (@correctroadh)
