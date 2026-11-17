---
title: "$$$ How I Discovered a Business Logic Vulnerability That Allowed Price Manipulation Through Shipping Parameters"
author: Zyad Ibrahim (@zyad_ibrahim333)
url: https://medium.com/@zyad_ibrahim333/how-i-discovered-a-business-logic-vulnerability-that-allowed-price-manipulation-through-59d4a8c36cea
date_published: 2025-11-24
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: not stated (amount not shown; triaged P3, bounty paid about 2 weeks later)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. Full article was publicly visible (not members-only). Navigation/recommendation boilerplate removed. -->

(2 min read, Nov 24, 2025. Tags: Cybersecurity, Bug Bounty Tips, Bug Bounty, Programming, Hacking)

[Arabic opening salutation and a Quran verse (Az-Zumar 52) omitted as non-technical.]

I'd like to share a simple yet impactful **business logic flaw** I recently found during a routine test.

While testing an online store, let's call it https://target.com, I was trying to figure out if I could tamper with product prices. None of the usual parameters reflected the actual item price, so direct manipulation didn't seem possible.

That's when I decided to approach it differently.

Instead of targeting the product price directly, I thought:

**"What if I try manipulating the shipping fees... or even remove the tax?"**

So I started intercepting requests until I found the following endpoint:

```
https://checkout.target.com/checkout-api/v2/order/place-order
```

And this is where things got interesting.

Inside the request, I noticed a parameter responsible for the shipping destination. It didn't contain any price fields to modify, but it _did_ contain the country/region used to calculate shipping costs.

So I tried changing the `shippingCountry` to a different destination, one that I knew had lower shipping fees.

[screenshot omitted]

You can probably guess what happened next...

**The shipping price dropped instantly during checkout (PayPal), while the actual shipping destination remained unchanged.**

The logic was fully client-controlled.

No server-side validation.

No recalculation on the backend.

Just... lower shipping fees and no tax at all.

After that, I took a short break, only six hours, then came back and wrote the report.

About a month later it got **triaged as P3**, and two weeks after that I received the bounty.

[screenshot of bounty omitted]

**Note:** I published a similar writeup earlier, but this one is a refined and improved version.
