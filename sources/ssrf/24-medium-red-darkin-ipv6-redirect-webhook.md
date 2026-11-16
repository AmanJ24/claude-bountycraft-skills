---
title: "A Real SSRF Story from HackerOne (Featuring IPv6 + Redirects)"
author: Red Darkin (@red_darkin), with madara_
url: https://medium.com/@red_darkin/a-real-ssrf-story-from-hackerone-featuring-ipv6-redirects-9aa5e2ad8c2e
date_published: 2026-04-15
date_scraped: 2026-11-16
platform: medium
disclosed_report: https://hackerone.com/reports/2301565
bounty: "$2,500"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim.
     Related candidate: row #9 (H1 report 2301565, same bug). -->

In the world of web security, even the most seemingly harmless features like webhooks can hide critical vulnerabilities. In this article, we take a deep dive into a **Server-Side Request Forgery (SSRF)** bug discovered in a webhook implementation. What made this case especially interesting is that exploitation was only possible by chaining **IPv6** behavior with **redirection** tricks. Here's a technical breakdown of how my good friend madara_ (https://hackerone.com/madara_) and I (https://hackerone.com/red_darkin) identified and exploited this vulnerability.

In modern applications, features like webhooks are everywhere. They enable automation, integrations, and real-time workflows.

[image: Vulnerable feature]

But they also introduce something dangerous:

> **You are letting your server make requests on behalf of the user.**

And that's exactly where things start to break.

This is the story of how a seemingly "secure" webhook implementation led to a **Server-Side Request Forgery (SSRF)** using a combination of **IPv6 encoding** and **redirect abuse**.

[image]

**First, What is SSRF (Quickly) ...**

A Server-Side Request Forgery happens when an attacker tricks a server into sending requests to unintended destinations, such as:

- Internal services (`127.0.0.1`)
- Private APIs
- Cloud metadata endpoints
- Admin interfaces

Instead of attacking directly, you let the server do the work for you.

### The Entry Point: A Webhook Feature

While testing a target on HackerOne, we found a webhook functionality. Nothing unusual. Until we noticed:

- It accepted user-controlled URLs
- It triggered server-side requests
- It had some filtering in place

At first glance, it looked hardened.

### Initial Behavior

The application had protections:

- Blocking internal IPs in decimal format (like `127.0.0.1`)
- Filtering common SSRF payloads
- Allowing redirection but still blocking internal IPs in decimal format

So... no direct SSRF.

But then we focus test something simple: **Redirects**

[image]

We discovered the backend followed redirects automatically but it did not re-validate the destination, this is a classic mistake, so instead of targeting internal resources directly, we tried:

http://attacker.com → redirect → internal resource (decimal) → Still blocked.

[image]

So we went deeper.

### The Breakthrough: IP Representation Tricks

Filters often block this:

`127.0.0.1`

But forget that the same IP can be written differently.

`[::ffff:7f00:1]`

This is basically encode IPv4 as IPv6... And the filter? didn't catch it.

[image]

Full Exploit Chain

This is the redirector file we used to check the vulnerability exists.

```php
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept");
header("Content-Type: application/json");
header("Location: http://[::ffff:a9fe:a9fe]");
?>
```

[image]

Moment when we realized the SSRF was possible.

Once we realized we had a window into the internal infrastructure, the obvious next step was clear: port scanning, but the system had other plans.

The webhook responses were painfully inconsistent... delays everywhere, no reliable timing, no clean signals. Open ports, closed ports... everything looked the same. Automation was useless.

So we adapted.

Instead of relying on tools, we went manual; slow, deliberate, and focused. We started probing well-known ports one by one, looking for anything that behaved differently.

It wasn't fast. But it worked.

[image]

**Open ports discovered:**

- 5000
- 5012
- 8090
- 8126
- 24220

This is where things get interesting. Internal services often expose:

- Debug endpoints
- Admin panels
- Internal APIs
- Metrics systems

[image]

[image]

Along the way, we also discovered that even after the redirect, it was still possible to bypass the filtering mechanisms using alternative IP representations like octal and IPv6.

To streamline the exploitation, we ended up using a payload like:

`https://ourserver/h1/redirector.php?url=http://[::ffff:7f00:1]:§5000§`

This bug wasn't about breaking a system. It was about understanding it better than it understands itself. The filter didn't fail because it was weak, it failed because it was incomplete.

## Timeline

- Jan 2, 2024: Report submitted
- Jan 3: "Need more info"
- Jan 8: Triaged
- Jan 9: "Please don't escalate further..."
- Jan 10: Severity downgraded
- Jan 16: $2,500 bounty
- Jan 30: Disclosure [Report: https://hackerone.com/reports/2301565]

More detailed slides about SSRF bypass Techniques: https://github.com/Red-Darkin/h1-Comunity-Colombia---Slides/blob/main/SSRF.pdf

## Credits

Big shoutout to **madara_** for teaming up on this one.

And remember It's never just about the bug, it's about how far you can push it.
