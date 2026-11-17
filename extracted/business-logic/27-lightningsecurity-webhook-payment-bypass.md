# Forged payment webhook marks the account as paid (webhook/IPN spoofing)

- **Vuln type / sub-variant:** Payment bypass via unverified server-to-server webhook (IPN spoofing); the merchant trusts a forged payment-provider notification
- **Target type:** Subscription site with a payment-provider webhook endpoint (e.g. `PUT /api/webhooks/stripe`), an endpoint the user never touches in the normal flow
- **Recon signal:** The site offered a monthly subscription, and internal API docs referenced a Stripe webhook endpoint. Webhooks are provider-to-server only, so hunters rarely test them, which makes them a soft spot.
- **Root cause:** The webhook handler parsed the JSON body and acted on `status:"success"` without verifying the request actually came from the provider (no signature/HMAC check, no IP allowlist). Stripe supports signature verification but documents it as a sidenote and its sample code omits it, so integrations often skip it.
- **Exploitation steps:**
  1. Find the webhook endpoint (search JS for "webhook"/"payment", check the org's GitHub, or guess `/api/stripe/webhook`, `/api/payments/webhook`, `/api/stripeWebhook`).
  2. Learn the provider's webhook body shape; an empty body errors.
  3. POST a forged body, e.g. `{"payment":{"status":"success","provider":"stripe"},"id":"..."}`.
  4. The server responds success and the account is marked as having a paid subscription.
- **Bypasses used:** Forging a server-to-server notification the merchant did not authenticate.
- **Impact:** Free paid subscription / entitlement without any real payment. Generalizes to any sensitive incoming webhook (not just payments) that is trusted by shape.
- **Bounty:** not stated.
- **Lesson / pattern:** Webhook endpoints are an under-tested payment sink. Any incoming webhook/IPN for a sensitive action must be verified by signature/secret or it can be forged. Cue: a subscription feature implies a payment webhook worth hunting.
- **Source:**
  - Title: Bypassing Payments Using Webhooks
  - Author: Jack Cable (Lightning Security)
  - URL: https://lightningsecurity.io/blog/bypassing-payments-using-webhooks/
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/business-logic/27-lightningsecurity-webhook-payment-bypass.md`

Tags: premium-bypass, subscription-bypass, missing-server-validation, assumption-flaw, free-services, financial-loss, methodology
