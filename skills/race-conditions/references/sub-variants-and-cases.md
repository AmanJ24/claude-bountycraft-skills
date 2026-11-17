# Race condition sub-variants and real cases

The taxonomy from the Kettle whitepaper (note 01) and PortSwigger topic (note 03), each with the real findings in this set. Note numbers refer to `../../extracted/race-conditions/`.

## 1. Limit overrun (the simplest, most common)
A single-use or limited action, redeemed/claimed many times in one parallel burst because the limit check and the action are not atomic.
- **Instacart coupon** (note 10): a promo code redeemed several times in parallel, stacking discounts (a $31.63 order); a second variant raced two different new-user codes for $50 instead of $30. Paid twice.
- **Cosmos/Starport faucet** (note 11): cap 11 tokens, 5 per request; 50 concurrent requests yielded 30 tokens. Critical 9.3, $5,000 ($2,500 + $2,500 collaborator).
- **Slack onboarding survey** (note 12): $100 credit granted multiple times; 2 parallel requests = $200; 3-4 parallel curl jobs raised the odds. Fix took ~1 year.
- **HackerOne retest** (note 16): a retest confirmed ~5 times with background curl gave duplicate payouts; one $500 duplicate actually paid; ~$2,100 net, Medium 4.2.
- **hacker101 flag submission** (note 17): a flag resubmitted ~70x (Race The Web) inflated points; first fix was partial.
- Labs: the PortSwigger limit-overrun coupon lab (notes 05, 06, 18) drops the price from ~1,337 to ~15-37 by stacking.

## 2. Multi-endpoint collision
Two different endpoints race so one lands in the other's sub-state.
- **Payment vs add-to-cart** (notes 03, 06): add the expensive item into the window between payment validation and order completion; balance ends negative. Queue two add-to-cart copies so a second gets another chance. Requires window alignment (connection warming / server-side delay).
- Keyed on the shared session cookie, which is why the collision is possible.

## 3. Single-endpoint collision
Two parallel requests from the same session with different values, where the server stores the two fields (e.g. target record and token) separately.
- **GitLab email change** (CVE-2022-4037, note 01): two parallel `POST /-/profile` setting `user[email]` to two addresses; the email sent to B carried a valid token for A. For a valid token the app must be on the "resend existing token" path. Using your own address plus a target-company address yields a **verified email you do not own**, which chains to invitation hijack and third-party account takeover via GitLab as OpenID IdP.
- **Omise team invites** (note 15): a 30-connection gated burst on `POST /team/memberships` created duplicate invites; `/test/links` had the same flaw.

## 4. Object masking (a limit-overrun side effect)
Parallel creation where the list view shows one object per key, so a benign object can be swapped for a privileged one later.
- **GitLab invitations** (note 01): 6 identical invites in parallel gave 1 success, 5 "already taken", and 2 emails; a low-privilege decoy invitation can be replaced by an admin-level one on revoke.

## 5. State corruption / TOCTOU
A collision leaves the record in an inconsistent, often irreversible state.
- **hacker101 group join** (note 14): 5 parallel joins added the same user twice; afterwards neither the group leader nor the user could remove them. Rated 2.4; the impact is the stuck state, not value.
- **Every.org follow** (note 13): 30 gated requests produced duplicate follows (multiple 201s); fixed by making the action idempotent. Low severity; the lesson is that a non-idempotent write is the tell.

## 6. Partial construction
Race an authenticated request against a half-built object whose security field is still uninitialized.
- Registration inserts the user row, then sets the API key in a later statement; a request in between reads the empty field. Match the uninitialized value: PHP `api-key[]=` is an empty array, Rails `param[key]` yields nil. Example `GET /api/user/info?user=victim&api-key[]=` (note 03).

## 7. Sub-state (hidden multi-step in one request)
One request walks through an intermediate state with weaker protection.
- **Login vs MFA** (note 03): the session is valid before `enforce_mfa` runs; race the login against a sensitive authenticated page to act in that window.

## 8. Time-sensitive
Two operations collide on a timestamp-derived value.
- A reset token built only from a high-resolution timestamp collides when two resets land on the same tick; precisely time two resets for different users to mint colliding tokens (note 03).

## 9. Deferred collision
No immediate response difference; the collision surfaces later because work is batched.
- An unnamed site (note 01) batch-processed emails, so two email-change requests sent up to ~20 minutes apart still collided. Spot via later anomalies (two emails to one address), not response diffs.

## 10. Rate-limit / OTP brute via race
Win the race so many guesses land before the attempt limit applies.
- A numeric one-time token with a 5-attempt limit: standard single-packet gives ~20-30 tries; **first sequence sync** (note 04) lands ~1,000 tries in one burst, a reliable one-time-token authentication bypass. The ceiling is the server's `SETTINGS_MAX_CONCURRENT_STREAMS`.

## Cross-skill boundary
Many of these overlap `business-logic` (the invariant being raced is a business rule) and `idor`/access-control (the state corrupted is ownership/role). When the core issue is the **timing window on a check-then-act**, it is a race and belongs here; when the same endpoint is exploitable with a single sequential request (no concurrency needed), it is the other class. The payment/wallet-topup race is cross-referenced from the `business-logic` payment-bypass material.
