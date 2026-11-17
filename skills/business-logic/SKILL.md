---
name: business-logic
description: Use when testing any feature that enforces a business rule, money, identity, quantity, step-order, one-time use, or plan/role, where the input is syntactically valid but can be made to violate the intended process (checkout/cart pricing, coupons and promos, subscription/plan gating, multi-step or payment workflows, OTP/2FA and signup flows, balance/credit ledgers, licensing, referrals/rewards). Covers recognizing logic flaws, enumerating invariants, the manipulation and workflow-abuse techniques, avoiding false positives, and writing up net impact. Research/methodology only: it does not send requests to live targets.
---

# Business Logic Vulnerabilities

Synthesized from 21 real bug bounty findings and references (HackerOne disclosed reports incl. a $5,000 Stripe race and a $3,250 OTP-takeover, PortSwigger's logic-flaws taxonomy and labs, OWASP's Business Logic cheat sheet, Intigriti research, practitioner writeups). Every pattern traces to `sources.md`. Disclosed bounties here ran from a few hundred dollars to $5,000; the biggest paid when a logic quirk became unlimited financial abuse or a takeover.

**The one idea:** every business rule the app relies on is an **invariant**: coupon used once per user, price equals the catalog price, balance never goes negative, step B only after step A, a free-tier account cannot use a paid role, a token only authorizes the object it was issued for. A business logic bug exists wherever an invariant is enforced **only by the UI/client, or nowhere, or at one entry point but not another**. The input is always syntactically valid, so scanners and WAFs never see it. You find these by naming the invariant and asking: **what on the server actually enforces this?** If the answer is "the UI" or "nothing," there is a bug.

**Scope rule:** methodology and guidance only. It does not fire requests at live targets. Real testing needs an authorized, in-scope program and accounts you control (two accounts, and where relevant two plans/tenants/roles). Logic testing can move real money, grant real entitlements, or claim real resources, so stay on test data and stop at proof (see `../../CLAUDE.md`).

## 1. Recognize it

A logic flaw lives at the point where the application **trusts a value, a sequence, or a client's self-report** instead of re-deriving or re-checking it on the server. The tell is a feature with a rule you can state in one sentence ("you can only redeem this once", "this costs $40", "only Owners manage secrets") whose enforcement you cannot see on the server.

Signals, roughly in order of how often they paid out in this set:
- A **request carries a money or quantity field**: `price`, `amount`, `total`, `subtotal`, `tax`, `shipping`, `discount`, `quantity`, `seats`, `credits`, `currency`. Any of these the client sends is a candidate (Upserve, Shipt, Krisp, WooCommerce, Zyad, Intigriti).
- A **client-supplied field decides entitlement or role**: `access_level`, `role`, `is_premium`, `plan`, a subscription-status boolean in a **response** (Lovable, Streamlabs, Curve, secrets-mgmt).
- A **one-time or limited action**: accept-offer, redeem-coupon, claim-bonus, use-referral, add-seat. Candidate for **replay then race** (Stripe, OWASP, Intigriti).
- A **multi-step flow** (cart -> pay -> confirm, signup -> OTP -> done): candidate for **skip/repeat/reorder and calling a later step directly** (PortSwigger labs, OWASP, bitpanic).
- A **verification token tied loosely to its target**: an OTP/`requestId` that may not be bound to the `resId`/user it was issued for (Eternal/Zomato).
- A **mandatory parameter** whose absence changes the code path: delete `code`/OTP/token entirely, not just empty it (hover.com signup).
- A **feature hidden from a role or plan in the UI**: the API behind it may not enforce the same limit (secrets-mgmt UI-vs-API, Lovable, Curve).
- **Value derived from observed events, not final state**: a ledger credited before an upstream operation is confirmed/settled (Coinbase reverted-transfer credit).

## 2. Where it hides

Feature classes that actually produced findings. Full list with example reports in `references/attack-surfaces.md`.

- **Checkout / cart pricing**: every money field in the order (price, quantity, tax, shipping, discount, currency). Negative quantity, decimal quantity, overflow, currency swap, shipping/country as a pricing input (Upserve, Shipt, Krisp, Zyad, WooCommerce).
- **Coupons / promos / referrals / rewards**: single-use offers, per-user limits, expiry, stacking, personal (birthday) coupons, free-trial resets (Stripe, OWASP, Intigriti, Curve).
- **Subscription / plan / role gating**: paid-only roles or features gated only in the UI or by a response flag (Lovable `access_level`, Streamlabs response flip, Curve, secrets-mgmt).
- **Multi-step and payment workflows**: skip the payment or approval step, call the completion endpoint directly, replay the gateway handoff (PortSwigger insufficient-workflow labs, WooCommerce, OWASP).
- **Payment-gateway integration (the dedicated payment-bypass surface)**: a checkout that hands off to Stripe/PayPal/PayU/Razorpay has four trust points the merchant may fail to re-verify, each its own test: the **client-sent price/amount/currency**, the **return URL** (treating `/payment/success?order_id=` as proof), the **server-side webhook/IPN** (forgeable if the signature/IP is not checked), and the **relayed gateway callback** (a captured real "success" callback, with its valid gateway hash, replayed for a new unpaid order). Plus a **confirm-payment race** on wallet top-ups (webhook note 27, taxonomy note 28, PayU callback-replay note 29).
- **OTP / 2FA / signup / claim flows**: omit the verification field, reuse a token across resources, reach the resource by a path that skips the gate (hover.com, Eternal/Zomato, HackerOne 2FA-collaborator).
- **Balance / credit / ledgers**: credit granted on an unconfirmed or reverted upstream event; refund logic that misses prior refunds (Coinbase, Intigriti).
- **Licensing / seats / billing**: double-processed payment callbacks, non-idempotent purchase handling, seats-vs-price rounding (Krisp, PortSwigger license).

## 3. Test methodology

Short version. Full step-by-step in `references/testing-methodology.md`.

1. **Learn the domain and map the intended workflow.** Logic bugs require understanding what the feature is *supposed* to do. Walk it as a normal user first; obscure domains (fintech, licensing, logistics) are less picked over.
2. **Enumerate the invariants.** Write the one-sentence rules: pricing, quantity/limits, one-time use, step order, uniqueness, ownership, role/plan, balance. For each, ask what enforces it.
3. **Attack the money fields.** Tamper price/amount/tax/shipping/discount; send negative, decimal, zero, and huge (overflow) values; swap the currency code while leaving the amount. Keep totals self-consistent if the server checks internal consistency (Upserve). See `references/manipulation-and-bypasses.md`.
4. **Attack entitlement and identity.** Replay a UI-locked call with the paid/privileged value; flip response booleans and re-click the privileged action; build a role-by-endpoint matrix and replay the high role's traffic as a low one.
5. **Attack one-time and limited actions.** Plain replay first, then race it (parallel/single-packet). After any fix, retest with concurrency; "check then act" patches often stay racy (Stripe).
6. **Attack the workflow.** Forced browsing: skip, repeat, reorder steps; call later/completion endpoints directly; delete a mandatory parameter (name and value); read errors from half-initialized state.
7. **Attack token binding.** Get a valid token/OTP for an object you own, then replay the verify step with another object's id (Eternal). Check that every token is bound to the user, resource, and action it authorizes.
8. **Apply the impact filter, then prove net benefit.** Map the result to confidentiality/integrity/availability; confirm the attacker actually ends up better off (paid less, got the entitlement, took the resource), end-to-end, before reporting. Stop at proof on test data.

## 4. Known manipulation / abuse patterns

The levers that turn a valid request into a rule violation. Summary; full catalogue with examples in `references/manipulation-and-bypasses.md`.

- **Parameter tampering.** Edit any money/identity/role field the client sends, including hidden and disabled fields (Shipt, Adobe, Lovable). If the server checks internal consistency, keep the totals balanced (Upserve).
- **Unconventional numeric input.** Negative quantity subtracts from the total (Upserve); decimal quantity exploits mismatched rounding between units-granted and units-billed (Krisp); huge values overflow to zero/negative; zero/empty for free.
- **Currency confusion.** Change the currency code while the amount stays the same, so 100 of a weaker currency is charged. (Distinct from a static per-currency price table, which is a pricing decision, not a bug: see section 5.)
- **Response manipulation.** Flip entitlement booleans or blank a status array in the response, then perform the privileged action; it only counts if a **backend** endpoint accepts the follow-up (Streamlabs, Curve).
- **UI-vs-API mismatch.** A role/plan limit shown only in the UI; call the backend endpoint directly as the low-privilege account (secrets-mgmt, Lovable).
- **Parameter deletion.** Remove a mandatory field entirely (OTP `code`, token, captcha) to reach an unvalidated code path (hover.com).
- **Forced browsing / step abuse.** Skip, repeat, reorder, or directly hit completion endpoints of a multi-step flow; exploit state that never expires (OWASP, PortSwigger).
- **Replay and race.** Re-run one-time actions; then race any check-then-act with parallel or single-packet requests (Stripe, OWASP).
- **Token unbinding.** Reuse a verification token/OTP issued for your own object against someone else's (Eternal).
- **Event-vs-final-state trust.** Trigger a credit/entitlement on an upstream event that later reverts or never settles (Coinbase).
- **Payment-gateway bypass levers** (full detail in `references/manipulation-and-bypasses.md`): forge the **webhook/IPN** (`{"status":"success"}` straight to `/api/webhooks/<provider>` when the signature/IP is unchecked, note 27); **forced-browse the return URL** (`/checkout/complete?status=paid&orderid=`) when reaching it is treated as payment; **replay a captured real success callback** (keep the gateway hash and ids, note 29) when the merchant trusts the relayed result instead of calling the gateway's verify-transaction API; swap the **currency** while the amount check passes.

## 5. Common false positives

- **Tampering only your own data / display.** Changing a price in the browser and seeing a lower number on the gateway page is not a bug if the merchant verifies the paid amount afterward. WooCommerce's PayPal IPN check marks the mismatched order invalid post-hoc (WooCommerce report, closed). Always confirm the order is actually **fulfilled at the lower price**, not just shown.
- **Static per-currency price tables ("currency arbitrage").** Paying in a cheaper listed currency via the legitimate selector is a pricing/business decision, not a vulnerability, and gets closed Informative (PortSwigger report). A real currency bug needs an attacker-controlled, mismatched currency vs amount (section 4).
- **No net benefit.** The logic changed but the attacker ends up worse off (e.g. a license merge that *reduced* seats). A logic bug only counts when it nets the attacker an advantage; demonstrate the advantageous end state, not just a changed field (PortSwigger license report).
- **Pure entitlement bypass with weak impact.** Using a paid feature without paying is real but programs often rate it low or dispute it; frame concrete business cost and cite precedent (Lovable, Streamlabs).
- **Intended scope.** A control the vendor deliberately applied to one actor and not another (2FA required for submitters, not invitees) may be ruled working-as-designed; argue against the control's stated purpose (HackerOne).
- **"Impact filter" fails.** If you cannot tie the quirk to confidentiality, integrity, or availability, it is likely not reportable (Intigriti). A wrong non-financial calculation on your own account usually is not.

## 6. Reporting

What made reports in this set land well (template in `references/reporting.md`):
- **Lead with the demonstrated net benefit.** "Paid $18.70 for a $33 order", "turned a $20k offer into ~$600k of fee-free processing", "claimed a restaurant listing I do not own". Show the attacker is concretely better off.
- **Name the exact invariant and the missing enforcement.** "Server checks that the total matches the lines but never validates quantities as positive or reprices from its catalog"; "verify endpoint checks the OTP but not that it was issued for this `resId`"; "role limit exists only in the UI".
- **Prove it end-to-end.** Carry the effect through checkout, settlement, or the privileged action, not just a changed field or a lower number on a page; use a test environment/test cards if production charges are involved (Krisp, WooCommerce lessons).
- **Quantify and scale.** Per-unit savings times volume, accounts createable, resources claimable. Sequential/unlimited turns medium into high.
- **Pre-empt the severity dispute.** For entitlement/business-decision classes, state the business cost and cite similar accepted reports; expect pushback and address the control's intended purpose.

## 7. Sources

See `sources.md` for the full list (title, author, URL, disclosed report, bounty) and `../../extracted/business-logic/` for the per-writeup notes each claim is drawn from. Deeper references:
- `references/attack-surfaces.md`: feature classes with example reports
- `references/testing-methodology.md`: step-by-step, invariant enumeration, the impact filter
- `references/manipulation-and-bypasses.md`: full technique catalogue with payload shapes
- `references/escalation-and-impact.md`: turning a quirk into demonstrated impact; severity signals
- `references/reporting.md`: write-up template
