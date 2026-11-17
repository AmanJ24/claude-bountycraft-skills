# Replaying a real gateway success callback to pay for a new order for free

- **Vuln type / sub-variant:** Payment bypass via replaying the client-relayed gateway success callback (response/callback manipulation, no server-side gateway verification)
- **Target type:** Government e-commerce/payment site on ASP.NET using PayU; the gateway result is relayed back through the client to merchant endpoints (/PayCallBack.aspx success, /PayFail.aspx failure)
- **Recon signal:** Clicking Back/cancel on the gateway page produced an interceptable POST back to the merchant (a failure callback). That the outcome is delivered via a client-relayed POST is the cue that the success callback might be forgeable or replayable.
- **Root cause:** The merchant marked an order paid based on the client-relayed PayU callback (its `status` plus the PayU `hash`), without calling PayU's transaction-verification API server-side. A previously valid success callback (with its genuine gateway hash) could be replayed to satisfy a different, unpaid order.
- **Exploitation steps:**
  1. Observe both callbacks: cancel -> `POST /PayFail.aspx status=failure...`; a genuine small payment -> `POST /PayCallBack.aspx status=success&unmappedstatus=captured...&hash=<valid>`.
  2. Note that naive edits of the failure request (flip status to success) are rejected, because the gateway `hash` and signed fields do not match a real success.
  3. Make one genuine minimal payment (5 rupees) and capture its real success callback (valid hash).
  4. Start a new payment, cancel it, and replace the cancellation body with the captured success callback, keeping `mihpayid`, `txnid`, `addedon`, `hash` from the real success.
  5. Server accepts it and marks the new order paid with no money charged; repeatable.
- **Bypasses used:** Capture-and-replay of a signed success callback; understanding the gateway's own status/error documentation to know which fields matter.
- **Impact:** Unlimited free transactions on a government payment portal (unauthorized transactions, financial loss). Self-rated high impact.
- **Bounty:** not stated (engagement, site anonymized).
- **Lesson / pattern:** When the gateway result is relayed through the client, the merchant must verify it server-side via the gateway's verify-transaction API, not trust the relayed status/hash. Test by capturing a real success callback and replaying/adapting it for an unpaid order. Flipping failure->success without a valid gateway hash will not work; a captured valid success will.
- **Source:**
  - Title: A High-Impact Payment Bypass on Government Website
  - Author: Amjad Ali (@amjadali110)
  - URL: https://amjadali110.medium.com/a-high-impact-payment-bypass-on-government-website-a-tale-of-business-logic-flaw-exploitation-b158feb7826a
  - Date: 2023-08-10
  - Disclosed report: none
  - Raw file: `sources/business-logic/29-medium-amjadali-gateway-callback-replay.md`

Tags: response-manipulation, parameter-tampering, missing-server-validation, assumption-flaw, financial-loss, free-services, order-of-operations
