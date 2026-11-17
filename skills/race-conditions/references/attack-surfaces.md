# Race condition attack surfaces (feature classes)

Where races actually lived in this set. Each class names the check-then-act gap, the signal, and example reports (note numbers refer to `../../extracted/race-conditions/`). Scanners cannot find these, so they are hunted by hand on multi-step and single-use features.

## Coupons / promos / gift cards / discounts
Gap: "is this code unused / under its redemption limit?" then "apply it", non-atomically.
- Redeem the same code many times in parallel to stack the discount below cost (Instacart promo, note 10; the PortSwigger limit-overrun lab, notes 05, 06, 18). nopCommerce CVE-2024-58248 (gift-card reuse, no order lock) is cited in note 05.
- Signal: a discount/code a normal request applies only once; a "per user" or "once" limit.

## Faucets / balances / withdrawals / credits
Gap: "balance/cap sufficient?" then "debit/grant", with the check and the write separated.
- A faucet capped at 11 tokens (5/request) gave 30 tokens from 50 concurrent requests, Critical 9.3, $5,000 (Cosmos/Starport, note 11).
- An onboarding survey credit of $100 was granted multiple times; 2 parallel requests gave $200 (Slack, note 12).
- Withdrawals / wallet top-ups: concurrent "confirm" requests double-credit a single payment (the wallet-topup race from the payment-bypass taxonomy, business-logic note 28).
- Signal: a per-user cap, a balance debit, a one-time grant.

## Payment vs cart (multi-endpoint)
Gap: the window between "payment validated" and "order finalized" on two different endpoints.
- Race an add-to-cart (or add a second one for another chance) into that window so the expensive item is added after validation; balance ends negative (PortSwigger multi-endpoint lab, notes 03, 06).
- Signal: a checkout flow where cart and payment are separate requests keyed on the same session.

## Login vs MFA / auth sub-states
Gap: a single request sets the session, then enforces MFA a step later.
- Fire the login in parallel with a request to a sensitive authenticated page to land in the logged-in-but-MFA-not-yet-enforced window (note 03).
- Signal: a multi-step login; any `session['user']` set before a `require_2fa` check.

## Email change / confirmation (single-endpoint)
Gap: the confirmation email's recipient and its token are read from different sources, or only one pending change is stored.
- Two parallel email-change requests make the token mailed for address B carry a valid token for address A, yielding a verified email you do not own, which chains to invitation hijack and OpenID account takeover (GitLab CVE-2022-4037, note 01).
- Signal: "we sent a confirmation link" flows; the site storing one pending email at a time (editing, not appending).

## Social / membership actions
Gap: "already following / already a member?" then "add", non-atomically.
- Duplicate follows from 30 gated parallel requests; fixed by making the action idempotent (Every.org, note 13).
- Duplicate team invites / seats (Omise invite, note 15).
- Two parallel joins add the same user twice so neither the leader nor the user can remove them (undeletable member, TOCTOU state corruption, hacker101 group, note 14).
- Flag/points resubmission inflates score (hacker101 flag, 70x via Race The Web, note 17).
- Signal: follow/invite/join/vote; "undeletable"/"duplicate" states.

## Registration / object creation (partial construction)
Gap: an object is inserted, then its security field (API key, role) is set in a later statement.
- Race an authenticated request against the uninitialized field with a value equal to the uninitialized state (PHP `api-key[]=` empty array; Rails `param[key]` -> nil). Example: `GET /api/user/info?user=victim&api-key[]=` (note 03).
- Signal: a registration/provisioning flow that returns an object before all fields are populated.

## Rate-limited OTP / token
Gap: "attempts remaining?" then "check code", with the counter not updated atomically.
- Win the race so hundreds of guesses land before the 5-attempt limit applies; with first-sequence-sync, ~1,000 attempts in one burst against a numeric PIN (notes 03, 04).
- Signal: a login PIN / 2FA code / one-time token with a low attempt limit.

## Workflow / payout duplication
Gap: a confirm/submit action processed more than once.
- A single retest confirmed ~5 times in parallel produced duplicate payouts, one $500 actually paid, ~$2,100 net (HackerOne retest, note 16).
- Firefox Monitor email-address limit exceeded (HackerOne #1913309, cited in note 06).
- Signal: any "confirm"/"submit"/"accept" that grants a reward or advances a workflow once.
