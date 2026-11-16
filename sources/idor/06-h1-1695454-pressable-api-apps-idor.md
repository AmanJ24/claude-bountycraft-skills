---
title: "IDOR in API applications (able to see any API token, leads to account takeover)"
author: bugra
url: https://hackerone.com/reports/1695454
date_published: 2022-09-08
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1695454
bounty: not stated (awarded, amount hidden)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR in API applications (able to see any API token, leads to account takeover)

Reported to: Automattic | Reporter: bugra | Reported: 2022-09-08 | Disclosed: 2022-11-01 | State: Resolved | Severity: critical

## Report body

## Summary:
Hi,

@ehtis, thank you for the test account. Here is a critical report. :)
On Pressable, we can create API applications at https://my.pressable.com/api/applications, and we can access many things using the API token via following the [API docs](https://my.pressable.com/documentation/api/v1)

I created an API application and tried to update it, I saw this request :

████████

As you can see there is an `application[id]` parameter that contains the application ID. I changed it to my second account's application ID and that API app moved to my account. So, there is an IDOR but it doesn't have a great impact because it just removes the API application from the victim's account.

So I tried to escalate its impact and I noticed if we remove all parameters except `application[id]` and `authenticity_token`, then send the request, the endpoint gives an error with `Name must be provided` and prints the given application ID's page. And, that page contains `Client ID` and `Client Secret`!

With this information, the attacker can make many actions on the victim's account. (https://my.pressable.com/documentation/api/v1)

## Steps To Reproduce:

  1. Go to https://my.pressable.com/api/applications and create an API app
  1. Click on the application and turn on your proxy program 
  1. Click `Update` and you will send a POST request to `/api/applications`
  1. In this request, change the `application%5Bid%5D` parameter's value to the target app ID, **then remove all parameters except `application%5Bid%5D` and `authenticity_token`**
  1. The page will give an error and you will see the victim app's page which contains `Client ID` and `Client Secret`
  1. Now, you can use these API credentials on the Pressable API.

Notes:
- API application IDs are sequential, so the attacker doesn't have to guess the IDs, s/he can access all applications
- The impact is critical because we can access many things via the API, that includes the "collaborator" endpoint https://my.pressable.com/documentation/api/v1#collaborator-bulk-create

## Impact

The attacker can access all API credentials using this vulnerability, and that leads to account takeover (via adding collaborator etc.)

Regards,
Bugra

Request screenshot is redacted on the public page ("████████"). The report names the request body parameter `application[id]` (URL-encoded `application%5Bid%5D`) sent via POST to `/api/applications` on my.pressable.com along with `authenticity_token`.

## Timeline

- 2022-09-08 16:11 UTC: bugra submitted the report to Automattic (opening: thanks to ehtis for the test account).
- 2022-09-08 19:24: vortfu (Automattic staff) changed status to Triaged: "The Pressable team has deployed a patch which should fix this issue. Can you please verify that Client IDs / Secrets for applications (not owned by you) are no longer leakable."
- 2022-09-09, bugra: confirms it is fixed.
- 2022-09-12: ehtis (Automattic staff) closed the report as Resolved: "Working on a bounty for the two IDOR issues you reported." Automattic rewarded a bounty (amount hidden).
- 2022-09-12: bugra requested disclosure (notes the first HackerOne bounty came from Automattic and that he reached first place in their program). Reminder 2022-11-01. ehtis agreed; disclosed 2022-11-01 22:46 UTC.

## Metadata (from report sidebar)

Reported to: Automattic. Severity: Critical (9 ~ 10). Weakness: IDOR. State: Resolved. CVE: none. Bounty: hidden on page.
