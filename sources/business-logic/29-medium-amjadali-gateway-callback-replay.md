---
title: "A High-Impact Payment Bypass on Government Website: A Tale of Business Logic Flaw Exploitation"
author: Amjad Ali (@amjadali110)
url: https://amjadali110.medium.com/a-high-impact-payment-bypass-on-government-website-a-tale-of-business-logic-flaw-exploitation-b158feb7826a
date_published: 2023-08-10
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: not stated (VAPT engagement, government site anonymized as xyz.gov.in)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. Medium chrome trimmed. -->

# Payment bypass via replaying the gateway success callback (Amjad Ali, PayU)

A government site (xyz.gov.in) used PayU as the payment gateway. Checkout (CheckoutPage.aspx) with a nominal amount (5 rupees) redirected to PayU. The merchant received the result via a client-relayed POST callback to one of three ASPX endpoints depending on outcome.

## Flow observed
- Cancel: clicking Back on the PayU page and confirming cancel sent `POST /PayFail.aspx` with `status=failure&unmappedstatus=userCancelled&...&hash=<...>&error=E1605&...`.
- Success (genuine 5-rupee payment, captured next day): `POST /PayCallBack.aspx` with `status=success&unmappedstatus=captured&mode=UPI&net_amount_debit=5.06&...&hash=<gateway hash>&field7=APPROVED...&bank_ref_num=...&error=E000`.
- The success and failure callbacks differed in endpoint (/PayCallBack.aspx vs /PayFail.aspx), extra params (additionalCharges, mode, bank_ref_num), and the PayU `hash`.

## Failed attempts
- First tries just edited the failure request (status=failure -> success, PayFail.aspx -> PayPaid.aspx, net_amount_debit, field9), but the server rejected them ("Your Payment is failure"), because the PayU `hash` and other gateway-signed fields did not match a real success.

## Working exploit
- Made a genuine 5-rupee payment and captured the real success callback (POST /PayCallBack.aspx with the valid PayU hash for that transaction).
- Then started a new 5-rupee payment and cancelled it, and in the cancellation request replaced the entire body with the captured successful callback, keeping `mihpayid`, `txnid`, `addedon`, and `hash` from the real success.
- The server accepted it and marked the new transaction paid without any money being charged. Repeatable.

## Root cause
The merchant trusted the client-relayed PayU callback (its status and the gateway hash) to mark an order paid, without server-side verification against PayU's transaction-verification API. A previously valid success callback could be replayed to satisfy a new, unpaid order.

## Lesson
Capture the real success callback; a merchant that fulfills based on the relayed gateway response (rather than calling the gateway's verify-transaction API server-side) can be defeated by replaying/adapting a known-good success callback. Understanding the gateway's own status/error docs is what unlocked it.
