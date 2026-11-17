---
title: "Bypassing Payments Using Webhooks"
author: Jack Cable (Lightning Security)
url: https://lightningsecurity.io/blog/bypassing-payments-using-webhooks/
date_published: unknown
date_scraped: 2026-11-17
platform: personal-blog
disclosed_report: none
bounty: not stated
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

# Bypassing Payments Using Webhooks (Jack Cable)

Payment webhooks are functionality not visible to the end user: payment providers (Stripe, Braintree) use them to notify the site about subscription/payment updates. All communication is provider -> server, so a user never touches the webhook endpoint, and hunters often never test it.

## Finding
Testing a site with a monthly subscription, the researcher found internal API docs referencing `PUT /api/webhooks/stripe`. An empty JSON body errored. After checking Stripe's webhook format, this body:
```
{ "payment": { "status":"success", "provider":"stripe" }, "id":"..." }
```
returned success, and the account was marked as having a paid subscription. The server trusted the forged webhook because it never verified the request actually came from Stripe (no signature check).

## Prevention by providers
- Braintree: library parses and validates the webhook automatically (secure by default).
- Stripe: supports signature verification but documents it as a sidenote; example code parses JSON directly with no signature check, so many integrations never verify.
- Recurly: HTTP Basic auth shared secret + IP allowlist (but an attacker can make their own Recurly account to send valid webhooks from allowed IPs).
- BitPay: states it does not sign IPNs, so payload should not be trusted outright.

## Discovery tips
- A monthly-subscription feature is the cue to look for payment webhooks.
- Search JS files for "webhook" or "payment"; check the org's GitHub for webhook references.
- Guess endpoint shapes: /api/stripe/webhook, /api/payments/webhook, /api/stripeWebhook, etc.

## Lesson
Any incoming webhook/IPN for a sensitive action (payment, subscription) must be verified (signature/secret), not trusted by shape alone. Not limited to payments: any incoming webhook notification should be verified.
