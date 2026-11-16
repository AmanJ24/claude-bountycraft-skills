---
title: "How I Found an IDOR Vulnerability in an Online Store (and Got Paid $50 for It)"
author: Nexus0xFault (0xnexus0xfault0x)
url: https://medium.com/@0xnexus0xfault0x/how-a-sneaker-order-led-me-to-an-idor-vulnerability-and-50-d3553a9f9136
date_published: 2026-10-28
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: $50
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# How I Found an IDOR Vulnerability in an Online Store (and Got Paid $50 for It)

Subtitle: How a simple sneaker purchase turned into a Broken Access Control finding

In this article I mention a friend who agreed to help me test this. To protect his privacy, I'll call him Brian.

## How it started

I was recently talking with Brian about a pair of sneakers I'd been wanting to buy. He recommended a new online store he'd just discovered, one that runs frequent discounts on clothing, and the exact pair I wanted happened to be on sale. I figured, why not.

I signed up, entered my details, and placed the order. While the package was on its way, I started thinking: since the store was brand new, there was a decent chance it had some security gaps. So I opened Dev Tools in Firefox and started watching how the site talked to its backend.

I'm not naming the store. The vulnerability was fixed after I reported it, but it feels more responsible not to call it out.

## The finding

When I opened my own order in my account, the site displayed it normally, with all the expected details.

(Image 2: Dev Tools Network showing the GET request for my own order: `GET https://[site]/api/orders/{order_uuid}`. Domain redacted.)

(Image 3: server's JSON response for that request, the normal output for viewing your own order.)

The order identifier was a UUID.

I wanted to see how the server would handle someone else's or a nonexistent identifier, so I swapped the UUID in the request for a random one. The server returned **404 Not Found**.

That raised a question: was this intentional protection (the server recognizes the order isn't mine and returns 404 to avoid revealing whether it even exists), or just a coincidence, meaning no order with that ID existed, and the server doesn't check ownership at all? A 404 on a random UUID doesn't prove anything on its own, I needed to test this against an **existing** order that belonged to someone else.

## Testing the hypothesis

Brian, the friend who'd recommended the store, was also registered there. I asked him for his order number, and he agreed to share it (**important: this was done with his explicit permission, I didn't brute-force or guess anyone else's data**).

I swapped his UUID into the request while using my own session, and the server returned **the full details of his order**:

- sneaker brand and model
- size and quantity
- order date and time
- delivery city and pickup point
- order total

The hypothesis was confirmed: the server only checked whether the order existed, not whether it belonged to the requesting user.

## Making it worse: no authentication required

Next, I checked whether authentication was required at all. I cleared every cookie on the site and repeated the request without a session. The response was identical: the order data was returned to a completely unauthenticated, unauthorized user.

That turns the finding from "a logged-in user can see other users' orders" into "anyone with an order ID can see anyone's order, no login required."

To confirm this outside the browser entirely, I opened PowerShell and sent a plain `curl` request straight to the endpoint, no cookies, no auth headers, nothing:

`curl.exe -i https://[site]/api/orders/{order_uuid}`

The server responded with **200 OK** and returned the full order data, exactly as it had in the browser. No session, no login, no token, just the raw UUID.

## Classification and risk assessment

This is an **IDOR (Insecure Direct Object Reference)** vulnerability, also known as **BOLA (Broken Object Level Authorization)**. It sits at the top of the OWASP API Security Top 10 (API1).

Two things stand out here:

- Order IDs are **UUIDs**, not sequential numbers, which makes mass enumeration essentially impractical. That lowers the odds of exploitation, but it doesn't eliminate the vulnerability: a UUID is not an authorization mechanism. If an identifier ever leaks (through an email, a screenshot, server logs, or browser history), the order becomes exposed to anyone who has it.
- The leaked data is **personal information** (delivery address, order contents), and accessing it requires no authentication whatsoever.

I rated the severity as **medium** (depending on the scoring model used, it could reasonably land higher). Large-scale exploitation isn't realistic, but the privacy impact is real.

## Responsible disclosure

Right after confirming the issue, I emailed the store's support team. My report included:

1. a short summary of the vulnerability
2. the affected endpoint and reproduction steps
3. what data was exposed
4. recommended fixes
5. a note that my testing was minimal and limited to Brian's order, which he'd consented to share

I didn't publish anything or take the finding any further until the team had a chance to respond.

## The outcome

Support thanked me for the report. They classified the vulnerability as non-critical, but offered a **$50 reward** as a thank-you. It ended up covering roughly half the cost of the sneakers I'd ordered in the first place.

## How to fix this

For developers who might be dealing with something similar:

- **Always verify object ownership server-side** on every request (`order.user_id == current_user.id`).
- **Require authentication** on every endpoint that returns personal data.
- Don't rely on unpredictable identifiers (like UUIDs) as a substitute for real authorization checks.
- Return a consistent response (`403` or `404`) for both unauthorized and nonexistent objects, so you don't leak extra information either way.

## What I took away from this

- Even a simple online store is worth checking for basic security hygiene.
- Dev Tools and a bit of curiosity are often enough to surface serious issues.
