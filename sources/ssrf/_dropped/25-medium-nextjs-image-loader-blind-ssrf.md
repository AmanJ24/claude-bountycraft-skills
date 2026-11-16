---
title: "When an Image Loader Turned Into a Backdoor: My Discovery of a Blind SSRF on a Blockchain Explorer"
author: Anshubind (@anshubind89)
url: https://medium.com/@anshubind89/%EF%B8%8F-when-an-image-loader-turned-into-a-backdoor-my-discovery-of-a-blind-ssrf-on-a-blockchain-94d33d6d5371
date_published: 2025-12-10
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: "not stated"
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim.
     Scrape note: no member-only marker appeared, but the body ends abruptly after fix recommendation #2 (blocklist bullets), no conclusion. Page said "2 min read", which roughly matches the length returned. Likely complete but short; treat as thin. -->

Sometimes the most dangerous vulnerabilities hide behind the most harmless-looking parameters.

And in this case, it wasn't an admin panel, a misconfigured server, or an exposed endpoint.

It was an **image URL parameter**.

A simple one.

On a testnet blockchain explorer.

And it allowed the backend to make requests _anywhere I told it to_.

This is the story of how a Blind SSRF quietly appeared inside a Next.js image optimization endpoint, and how I uncovered it.

## The Setup: A Normal Recon Session

I wasn't expecting anything dramatic.

I was casually testing a public blockchain testnet explorer, you know, clicking around, viewing blocks, scanning JS files, the stuff we all do when curiosity kicks in.

Then I spotted something familiar:

```
/_next/image
```

Next.js uses this endpoint to optimize images.

And image endpoints often accept a URL parameter to fetch remote images.

The question that always follows is:

> "Are they validating what URLs they fetch?"

So I tried the obvious test.

## Crafting the Payload

I replaced the image URL with my Burp Collaborator domain:

```
/_next/image?w=16&q=50&url=https://<collaborator>.burpcollaborator.net
```

Hit enter.

The explorer page didn't show any meaningful output, expected for blind SSRF.

But a few seconds later...

My Burp Collaborator pinged back.

That was the moment.

The server, not my browser, had reached out to my listener.

This wasn't speculation.

This wasn't a browser preview.

This was a backend system performing an outbound request based entirely on user input.

A textbook **Blind SSRF**.

## Why Blind SSRF Is Dangerous

Even on a testnet, this kind of vulnerability matters, because the behavior is often shared with production systems.

Blind SSRF can be used to:

## Internal Network Scanning

Attackers can map internal services:

- Redis
- MongoDB
- Internal dashboards
- Hidden API hosts

## Cloud Metadata Access

In cloud environments, SSRF can allow access to metadata endpoints:

```
http://169.254.169.254/
```

Which can leak:

- Access tokens
- IAM credentials
- Instance metadata

## Server Abuse

An attacker could:

- Bounce attacks through the victim's server
- Attempt exploit delivery
- Target private services unavailable to the public

Even a testnet SSRF is a canary in the coal mine.

If it exists in staging/test, there's a real chance the same code path exists in production.

## Reporting the Issue

I put together a high-level report:

- Vulnerable endpoint
- Verified collaborator interaction
- Impact analysis
- Safe reproduction steps
- Suggested mitigations

No internal logs.

No sensitive data.

No exploit details beyond what was necessary for validation.

Just clean, responsible disclosure.

I reached out to the security team with everything they needed to reproduce and fix the issue.

A few days passed.

I followed up, politely, professionally, and awaited the official triage update.

## How It Should Be Fixed

Here are the same recommendations I shared with them:

## 1. Allow-list Only Trusted Image Domains

Never allow arbitrary URLs.

## 2. Reject or Sanitize User-Controlled URL Values

Block:

- Private IP ranges
- Loopback addresses
- Metadata endpoints
