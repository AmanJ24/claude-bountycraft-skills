# Business logic manipulation / abuse catalogue

The levers that turn a syntactically valid request into a violation of the intended rule. Each entry: the lever, how to apply it, and the example (note numbers in `../../extracted/business-logic/`). Payload shapes are representative; use benign test data on authorized targets only.

## Parameter tampering (money, identity, role)
Edit any field the client sends, including hidden and disabled form fields.
- Money: `price`, `amount`, `total`, `tax`, `shipping`, `discount` to a tiny value or zero (notes 03, 04, 06).
- Entitlement: `access_level`, `role`, `is_premium`, `plan`, `tenant_id` to a privileged value (note 14, Lovable set `access_level` write -> read).
- Internal-consistency defense: if the server only checks that the grand total matches the submitted lines, recompute the total/tax so your tampered lines still balance (note 01, Upserve).

## Unconventional numeric input
- **Negative quantity**: add a line with `quantity = -1` so its value subtracts from the order (note 01). Also try negative `amount` on transfers (balance check passes, funds flow the wrong way) (note 23).
- **Decimal quantity**: send `1.1` / `1.9` to a seats/quantity field; look for a ceiling on units-granted but a floor on units-billed (note 15, Krisp).
- **Integer overflow**: send a huge price/quantity so the backend wraps to negative or zero (note 05, Intigriti).
- **Zero / empty**: for free items or to skip a charge.

## Currency confusion
- Change the `currency` code (e.g. USD -> INR/JPY) while leaving `amount` unchanged, so a weaker currency's 100 units is charged and forwarded to the gateway (notes 05, 08).
- Not to be confused with a static per-currency price table, which is a pricing decision, not a bug (note 07; see SKILL.md section 5).

## Response manipulation
- Add proxy match-and-replace rules on **response** bodies: entitlement `false -> true`, `"blocked":true -> false`, or blank an "already selected" array (notes 13, 16).
- Only a finding if a backend endpoint accepts the follow-up privileged action. Click through to the real action (redeem, RTMP key, re-select) and quantify it (note 13).

## UI-vs-API mismatch
- A role/plan limit shown only in the UI: learn the API shape from a proxy, then call the backend endpoint directly as the low-privilege account (note 25, secrets-mgmt; note 14).
- Build a role-by-endpoint matrix from the highest role's captured traffic and replay as lower roles / unauthenticated.

## Parameter deletion
- Remove a mandatory verification field entirely, the key not just the value, so a conditional server check never runs: delete `code`/OTP/token/captcha (note 19, hover.com). Try: delete key, null value, empty string, each separately.

## Forced browsing / workflow abuse
- Call later or completion endpoints directly without the prerequisite step (notes 23, 12).
- Skip, repeat, reorder, and go back in a multi-step flow; hit the same URL with different parameter sets.
- Exploit half-finished states that never expire, and discount thresholds not re-checked after cart edits (add items to pass a threshold, then remove them) (note 23).

## Replay and race (check-then-act)
- Plain replay a one-time action N times first (note 11 applied 30 discounts by replay alone).
- Then race: parallel requests or an HTTP/2 single-packet / gated release to beat a non-atomic "already done" check (notes 11, 12). Targets: balance debit, single-use coupon, limit counter, slot/seat booking, unique registration.
- Always retest with concurrency after a fix.

## Token unbinding (cross-resource reuse)
- Obtain a valid token/OTP for an object you own, then replay the verify step with another object's id (note 20, Eternal: OTP+`requestId` valid for the attacker's `resId` accepted for a victim's).
- Generalizes to any `requestId`/token/nonce that authorizes an action: check it is bound to user + resource + action, and re-checked on use.

## Event-vs-final-state trust
- Trigger a credit/entitlement on an upstream event that can revert or is not yet settled: a reverted smart-contract internal transfer still credited off-chain (note 09, Coinbase); a payment callback fired twice (note 18).

## Mass assignment (adjacent)
- Add unexpected privileged fields to a create/update request (`role=admin`, `permissions=[*]`) the API did not allowlist (note 26, bitpanic). Overlaps the `idor` skill's mass-assignment section.
