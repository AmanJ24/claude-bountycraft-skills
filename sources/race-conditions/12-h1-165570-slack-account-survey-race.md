---
title: "Race Condition in account survey"
author: cablej
url: https://hackerone.com/reports/165570
date_published: 2016-09-03
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/165570
bounty: not stated (bounty awarded by Slack, amount hidden on report)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Scrape status: FULL (complete report body and timeline rendered; bounty amount hidden by HackerOne). Presigned attachment URLs replaced by [REDACTED]. Request cookie was already elided ("Cookie: ..."); the `crumb` CSRF value in the sample body is replaced with [REDACTED]. Code block reconstructed from rendered output.

Report #165570: Race Condition in account survey
Reported to: Slack. Reported by: cablej. Reported on: September 3, 2016, 8:44pm UTC. Disclosed: November 12, 2017, 11:25am UTC.
Status: Resolved. Severity: No rating. Weakness: Violation of Secure Design Principles. CVE: None. Bounty / Total reward: Hidden.

## Summary by Slack

@cablej found a race condition in our account creation survey, which allowed a user to receive extra Slack credits. We investigated and fixed the issue. Thanks for the report @cablej!

## Report body (cablej, September 3, 2016, 8:44pm UTC)

There exists a race condition in the beginning survey, allowing a user to get $100 in credit multiple times. In my example, I made 2 asynchronous requests, and was credited with $200.

POC:

1. Create a new slack team.
2. Set your password, and find the account creation survey.
3. Complete the survey, and intercept the request using a proxy such as BurpSuite.
4. Repeat the request asynchronously, such as in the command line by executing `(command) & (command)`.
5. The survey will be credited to your account multiple times. See the attached screenshot.

Please let me know if you need any more information.

Attachment F117127: Screen_Shot_2016-09-03_at_3.44.02_PM.png [REDACTED presigned URL]

## Timeline

**bugtriage-josh** changed the status to **Needs more info**. (September 6, 2016, 7:39pm UTC): Thank you for your report. Can you be a bit more specific about what request you are issuing and additionally where it is located in the UI? I created a team as in your reproduction steps, but didn't see a link to a survey. Can you clarify what specific commands are you performing so that we may replicate your results? Thanks, and good luck with your future bug hunting.

**cablej** changed the status to **New**. Updated September 6, 2016, 10:11pm UTC:

Ok, _finally_ found where the survey is offered :) It's after you set your password, which might only sometimes happen, which would explain why you hadn't seen it (and neither had I when I tried just now), but the URL still works: https://yourteam.slack.com/account/reset/complete.

More detailed POC:

1. Visit https://yourteam.slack.com/account/reset/complete, which gives you the option to take the survey.
2. Complete the server, and intercept the request to complete. It should look sort of like this:

```
POST /survey/6-23387113491-bed6344a95 HTTP/1.1
Host: slack.com
Content-Type: application/x-www-form-urlencoded
Origin: https://slack.com
Cookie: ...
Connection: close
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/601.5.17 (KHTML, like Gecko) Version/9.1 Safari/601.5.17
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Content-Length: 407

done2=1&crumb=[REDACTED]&referral_options%5B%5D=Airport&referral_options%5B%5D=Radio&referral_options_other=&interest_options%5B%5D=Works+seamlessly+across+all+devices&interest_options%5B%5D=Improve+productivity&interest_options_other=&software_options%5B%5D=Google+Hangouts%2FChat&software_options%5B%5D=Yammer&software_options_other=&company_size=1-9&company_industry=Consumer+Goods
```

3. Repeat the request multiple times, such as in the command line. In BurpSuite or Chrome developer tools, copy the request as a curl command, and execute:

`(command) & (command) & (command)`

I found that sending 3-4 requests increases the likelihood that multiple will be accepted.

After this, I had 2 survey completions in my account, both for $100.

Hope this helps!

**bugtriage-josh** (September 7, 2016, 6:35pm UTC): Thanks for the additional information @cablej. We werre able to reproduce this issue. Our investigation is ongoing and we'll be reaching out to you once we know more.

**maxfeldman** changed the status to **Triaged**. (September 8, 2016, 12:27am UTC): @cablej triaged!

**cablej** (October 13, 2016, 11:43pm UTC): Hi @maxfeldman, Any update on this report? Thanks, Jack

**maxfeldman** (October 17, 2016, 10:22pm UTC): Hi @cablej, This bug is still in their queue to be worked on. Thank you! Max

**cablej** (April 2, 2017, 4:56pm UTC): Hey @maxfeldman, Any update on this report? Thanks!

**maxfeldman** (April 10, 2017, 8:39pm UTC): Hey @cablej, This is still in the team's queue. Thanks! Max

**cablej** (September 10, 2017, 11:23pm UTC): Hey @maxfeldman, Is there any progress on this bug? I believe a year is more than a reasonable amount of time to patch a vulnerability. Thanks.

**cablej** requested mediation from HackerOne Support. (September 29, 2017, 7:57am UTC)

**maxfeldman** (October 4, 2017, 11:50pm UTC): Hey @cablej, Apologies for the delay; we were at a conference. I'm escalating this bug. Thanks, Max

**maxfeldman** (October 9, 2017, 10:16pm UTC): Hey @cablej, We believe we've fixed this issue - are you still able to reproduce? Thanks!

**cablej** (October 9, 2017, 10:36pm UTC): Hey @maxfeldman, I don't have an account with an available survey, so I'm currently unable to re-test. Thanks.

**maxfeldman** closed the report and changed the status to **Resolved**. (October 13, 2017, 7:25am UTC): Sounds good @cablej! We've tested and believe this issue is resolved.

**Slack** rewarded cablej with a bounty. (October 13, 2017, 7:25am UTC): Thank you for the report and for your patience! The Slack Security Team

**cablej** requested to disclose this report. (October 13, 2017, 11:25am UTC): Great, thanks!

This report has been disclosed. (November 12, 2017, 11:25am UTC)

Participants: cablej, maxfeldman, bugtriage-josh.
