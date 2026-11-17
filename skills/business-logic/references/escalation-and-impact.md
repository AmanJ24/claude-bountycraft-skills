# Business logic escalation and impact

A logic quirk is only worth reporting when it nets the attacker a concrete advantage and you can prove it. This reference covers the impact filter, how to climb from "a field changed" to demonstrated impact, and how the set was rated. Authorized, test-data, stop-at-proof only.

## The impact filter (apply before escalating or reporting)
Ask which of confidentiality, integrity, availability the quirk breaks (note 08, Intigriti):
- **Confidentiality**: you read another account's/tenant's data (orders, reports, listings).
- **Integrity**: you pay less and the order is fulfilled, grant yourself an entitlement, change state you should not.
- **Availability**: you block a real user (register their email first, exhaust a limited resource).
If none applies, it is likely not reportable: editing only your own data, a wrong non-financial calculation on your own account, or a cosmetic display change (notes 08, 13 early-triage).

## Prove net benefit, end to end
A business logic bug counts only when the attacker ends up **better off**, demonstrated through to the real effect:
- Not "the gateway page showed $0" but "the order was placed/fulfilled at the lower price" (note 04 failed this: IPN re-validated).
- Not "the licenses page changed" but "I gained seats/time" (note 18 failed this: the account ended up with fewer seats).
- Not "the displayed price dropped" but "checkout/settlement charged the lower amount", verified on a test environment with test cards if needed (note 15).

## The escalation ladder (quirk -> demonstrated impact)
1. **Field accepted**: the server took your tampered/odd value without error.
2. **State changed**: the stored order/role/balance/entitlement reflects it.
3. **Benefit realized**: you paid less and received goods, used the paid feature, claimed the resource, read the victim's data.
4. **Quantified**: per-unit savings times volume; number of accounts/resources affected.
5. **Scaled / unlimited**: the action repeats (replay/race), ids are sequential, or every tenant is affected. This is where medium becomes high/critical.

## Worked escalations from the set
- Pay-less chains: negative quantity -> order accepted at reduced total -> restaurant forced to resolve out-of-balance orders (note 01).
- One-time -> unlimited: single accept endpoint -> plain replay 30x -> ~$600k fee-free processing -> after fix, race still succeeds ~23/30 (note 11, $5,000).
- Token unbinding -> takeover: OTP for own resource -> verify against victim `resId` -> own a listing you do not control, Critical 9.1 (note 20, $3,250).
- UI-vs-API -> privilege escalation: Editor on free tier -> direct API secret creation -> Owner-only action, CVSS 8.3 (note 25, $1,500).
- Response flip -> free goods: subscription booleans true -> redeem button appears -> real reward coupon + stream keys, one per account, scalable by signup (note 13).
- Event-vs-state -> cash-out: reverted contract transfer still credited -> inflate exchange balance -> withdraw funds that never existed, Critical (note 09).

## Severity signals (how the set was rated)
- Unlimited financial abuse, cross-account takeover, or ledger inflation: high/critical (notes 11, 20, 09).
- Pay-less with real fulfillment, cross-tenant read: medium/high (notes 01, 06, 15).
- Pure entitlement bypass (paid feature for free) with no data/financial impact shown: often medium or disputed (notes 13, 14).
- No net benefit, self-data only, or a vendor "business decision"/intended-scope ruling: informative/N-A (notes 04, 07, 18, 21).

## Cross-skill note
Business logic overlaps its neighbours: a client-controlled object id that reads another account is `idor`; an unfiltered privileged field on a write is mass assignment; a check-then-act you win in parallel is a race condition (its own class). When the core issue is object ownership rather than a business rule, use the `idor` skill. When it is a timing window on a single-use action, treat the race as the primary finding (see note 11).
