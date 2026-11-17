---
title: "Race condition in performing retest allows duplicated payments"
author: cablej
url: https://hackerone.com/reports/429026
date_published: 2018-10-26
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/429026
bounty: "$2,100 net per a staff internal-style comment ($2,500 bounty total - $500 retest total + $100 retest initial); H1 sidebar shows amounts hidden"
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #429026. Reported to HackerOne (security program) on October 26, 2018, 1:04am UTC by cablej (Jack Cable). Status: Resolved. Severity: Medium (4.2). Weakness: Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition'). CVE: None. Disclosed December 27, 2018. Participants: cablej, jobert, d0nut, harisec, johnny, abze. Note: the first scrape returned only the sidebar; a retry with maxAge 0 returned the full body below.

## Summary

There exists a race condition in performing retests. By executing multiple requests to confirm a retest at the same time, a malicious user is paid multiple times for the retest. This allows for stealing money from HackerOne, which could go unnoticed by both HackerOne and the attacker (me).

## Steps to Reproduce

1. Receive a retest request email from HackerOne.

2. Intercept the request to retest the email. Right click the request in Burp Suite and select `Copy as curl command`.

3. Execute the request on the command line in the form `(request) & (request) & ...`. In testing, I executed the command 5 times.

4. Scroll to the bottom of https://hackerone.com/settings/bounties. The payment will appear under the `Retest payments` sections and may be repeated.

5. Wait a few weeks. If successful, a callback from HackerOne will be received (in this case from @michiel): [screenshot F366191: Screen_Shot_2018-10-25_at_5.58.17_PM.png, signed URL removed]

6. Check your bank account statements. Observe that a $500 payment was sent from HackerOne 2 weeks ago, demonstrating that the race condition was successful: [screenshot F366192: IMG_3755.png, signed URL removed]

## Impact

This allows an attacker to exploit the retesting feature to steal many times more money. Given that this went unnoticed by both the attacker and HackerOne for over 2 weeks, this has the potential to be exploited multiple times to steal money from HackerOne.

## Timeline

**jobert (HackerOne staff)** added weakness "Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition')" and removed weakness "Business Logic Errors". (October 26, 2018, 11:35pm UTC)

jobert updated the severity to medium (4.2). (October 26, 2018, 11:35pm UTC)

**jobert** closed the report and changed the status to **Resolved**. (October 26, 2018, 11:37pm UTC)

Hi @cablej - haha, well done! At first sight, when I noticed 5 internal activities on that report, it didn't look like the attack actually worked. We noticed it because the billing overview broke due to a data migration removing the redundant activities. According to engineering, this should be fixed now. We'll issue a retest for this particular report for you and others to retest this. Thanks a bunch!

**HackerOne rewarded cablej with a bounty.** (October 26, 2018, 11:39pm UTC)

$2,500 (bounty total) - $500 (retest total) + $100 (retest initial) = $2,100.

abze, johnny, harisec, d0nut joined this report as participants. (October 27, 2018)

**cablej posted a comment.** (October 30, 2018)

Confirmed that I only received 1 $100 payment this time.

**d0nut posted a comment.** (October 30, 2018)

Same here

cablej requested to disclose this report. (December 23, 2018)

jobert agreed to disclose this report. (December 27, 2018)

This report has been disclosed. (December 27, 2018, 12:12pm UTC)

## Metadata sidebar

Reported on October 26, 2018, 1:04am UTC. Reported by cablej. Reported to HackerOne (Managed). Severity Medium (4.2). Weakness: Race Condition (CWE-362). CVE: None. Bounty/Retest/Total reward: Hidden on page. Account details: None.

Capture status: FULL (after retry; images not captured).
