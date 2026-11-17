# Payment gateway bypass: the five-method taxonomy mapped to the checkout flow

- **Vuln type / sub-variant:** methodology/taxonomy (payment-gateway bypass)
- **Target type:** E-commerce checkout integrating a payment gateway (Stripe/PayPal/Razorpay): client, merchant server, and gateway, connected by a client Return URL and a server-side webhook
- **Recon signal:** Map the six-step flow (initiation, order creation, payment request, transaction, verification/callback, fulfillment) and find which step the merchant trusts without re-verifying: the client price, the return URL, the webhook, the currency, or the confirm-payment race.
- **Root cause:** The merchant marks an order paid on a "success" signal it does not independently verify against the gateway (amount, currency, authenticity, or idempotency).
- **Exploitation steps (testable patterns):**
  1. Parameter tampering: set the client-sent `amount` to 0.01; works if the backend checks only "successful", not that the paid amount equals the DB price.
  2. Return-URL / client callback: if reaching `/payment/success?order_id=123` is treated as proof, skip the payment page and hit the success URL directly (`/checkout/complete?status=paid&orderid=8872`), using your own pending order id.
  3. Webhook / IPN spoofing: POST the gateway's notification body straight to the merchant webhook if there is no signature check or IP allowlist (`{"event":"payment.captured","data":{"order_id":"MY_PENDING_ORDER","status":"completed"}}`).
  4. Currency switching: the amount is validated but not the currency; change GBP to IRR/VND so `paid_amount==100` passes but the real value is a fraction of a cent.
  5. Wallet top-up race: fire many simultaneous confirm-payment requests so concurrent reads of the balance credit one payment multiple times.
- **Bypasses used:** Parameter tampering, forced-browse to success URL, webhook forgery, currency swap, concurrency.
- **Impact:** n/a (reference). States payment bypass is almost always Critical: revenue loss, inventory exhaustion, PCI-DSS loss.
- **Bounty:** n/a (reference).
- **Lesson / pattern:** For any payment integration, treat all four "trust points" (client price, return URL, webhook, currency) plus the confirm-payment race as separate tests. Defenses: reprice from the DB, call the gateway's verify-transaction API server-side, verify webhook signatures + IP + idempotency, and checksum form data.
- **Source:**
  - Title: What is Payment Gateway Bypass? Ways to Exploit, Examples and Impact
  - Author: Nadeem Ahmad (Jsmon Blog)
  - URL: https://blogs.jsmon.sh/what-is-payment-gateway-bypass-ways-to-exploit-examples-and-impact/
  - Date: 2026-03-09
  - Disclosed report: none
  - Raw file: `sources/business-logic/28-jsmon-payment-gateway-bypass-taxonomy.md`

Tags: methodology, taxonomy, reference, defense, parameter-tampering, price-manipulation, workflow-bypass, currency-confusion, missing-server-validation, order-of-operations, financial-loss, free-services
