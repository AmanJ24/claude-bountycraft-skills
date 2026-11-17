---
title: "What is Payment Gateway Bypass? Ways to Exploit, Examples and Impact"
author: Nadeem Ahmad (Jsmon Blog)
url: https://blogs.jsmon.sh/what-is-payment-gateway-bypass-ways-to-exploit-examples-and-impact/
date_published: 2026-03-09
date_scraped: 2026-11-17
platform: other
disclosed_report: none
bounty: not applicable (reference / taxonomy)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. Nav/footer chrome trimmed. -->

# Payment Gateway Bypass: taxonomy (Jsmon)

## The standard payment workflow (for locating the gap)
Three parties: Client (browser), Merchant Server (the shop), Payment Gateway (Stripe/PayPal/Razorpay).
1. Initiation: user clicks Checkout.
2. Order Creation: merchant calculates total, creates an order id.
3. Payment Request: merchant sends order details (amount, currency, order id) to the gateway, via client-side redirect or server-side API call.
4. Transaction: user enters card details on the gateway page.
5. Verification & Callback: gateway responds to the merchant, via a client-side Return URL and a server-side Webhook.
6. Fulfillment: on a "Success" signal, merchant marks the order paid and ships.
A bypass interferes with any of these so the merchant believes payment succeeded when it did not.

## Common exploit methods
1. Parameter tampering (price manipulation): merchant trusts a client-sent price; intercept and set amount=0.01. Works if the backend only checks "payment successful" and not that the paid amount matches the DB price.
2. Return-URL / client-side callback manipulation: if the merchant treats reaching `/payment/success?order_id=123` as proof of payment, skip the payment page and navigate directly to the success URL (guess or use your own pending order id), e.g. `/checkout/complete?status=paid&orderid=8872`.
3. Webhook / IPN spoofing: webhooks are the server-to-server source of truth. If the merchant does not verify the signature (HMAC header) or allowlist the gateway IPs, forge the webhook with curl: `POST /api/webhooks/paypal` with `{"event":"payment.captured","data":{"order_id":"MY_PENDING_ORDER","status":"completed"}}`.
4. Currency switching: the system validates the amount but not the currency; change GBP to a far weaker currency (IRR, VND) so paid_amount==100 passes but 100 VND is a fraction of a cent.
5. Race conditions in wallet top-ups: send many simultaneous "confirm payment" requests (Turbo Intruder) so multiple threads read the same balance before any writes, crediting a single payment multiple times.

## Impact
Payment bypass is almost always Critical: direct revenue loss, inventory exhaustion (scripted free "purchases"), PCI-DSS compliance loss, reputational damage.

## Prevention
- Never trust client-side data: pull the price from the DB by item_id, not from a client `price` param.
- Server-side verification: before marking paid, call the gateway API (GET /v1/payments/{id}) to verify status and exact amount. Never rely on the success redirect.
- Secure webhooks: signature verification (recalculate the HMAC with your secret), IP allowlisting, and idempotency keys (same transaction id cannot be processed twice, which also stops the race).
- Cryptographic hashes for form data: hash amount+currency+order_id+secret and re-hash on return to detect tampering.
