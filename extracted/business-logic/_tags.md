# Canonical tags (Business Logic)

Normalized 2026-11-17. Use these spellings in `Tags:` lines.

- Sub-variant (price/value): price-manipulation, negative-quantity, fraction-rounding, integer-overflow, currency-confusion, parameter-tampering, balance-manipulation, discount-abuse, coupon-stacking, unlimited-redemption
- Sub-variant (access/flow): workflow-bypass, state-machine, order-of-operations, premium-bypass, subscription-bypass, license-logic, otp-bypass, 2fa-bypass, privilege-logic, mass-assignment
- Mechanism: client-trust, ui-vs-api, response-manipulation, missing-server-validation, assumption-flaw
- Impact: financial-loss, free-services, account-takeover, privilege-escalation, data-exposure
- Meta: methodology, taxonomy, reference, defense

Notes:
- `assumption-flaw` = the generalized root cause (a developer assumption the attacker violates); nearly every note carries it.
- `missing-server-validation` = the server trusts a client value or never recomputes/rechecks; the other near-universal tag.
- References (05, 08, 12, 23, 26) carry methodology/taxonomy/reference plus the sub-variants they cover.
- Partial sources: 09 (Coinbase), 10 (Inflection) have limited public detail.
- `07` (currency arbitrage) and `04` (WooCommerce IPN) are FALSE-POSITIVE / rejected examples: use in SKILL.md section 5.
- `18` (license-expiry) is a "no net benefit" example: logic changed, but the attacker ended up worse off.
