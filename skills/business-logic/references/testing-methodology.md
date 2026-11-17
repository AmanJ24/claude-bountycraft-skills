# Business logic testing methodology (step by step)

Research/methodology guidance. Against a real target, only proceed under authorized, in-scope conditions, on accounts and test data you control. Logic testing can move real money, grant real entitlements, or claim real resources, so prove on test data and stop at proof (see `../../CLAUDE.md`).

## Step 1 - Learn the domain and map the intended workflow
- Use the feature as a normal user, end to end, before testing. You cannot break a rule you have not understood.
- Read the docs/help for unfamiliar domains (licensing, fintech, logistics, loyalty). Obscure business domains are less picked over and pay better (note 23).
- Draw the happy-path state machine: the steps, who may do each, and what each one changes.

## Step 2 - Enumerate the invariants
Write each business rule as a one-sentence invariant, then name what enforces it. If the answer is "the UI" or "nothing", that is a lead (note 12).
- Pricing: "amount charged = catalog price for the real items/destination".
- Quantity/limits: "quantity is a positive integer in range"; "coupon once per user"; "3 retailers on free tier".
- Step order: "payment before fulfillment"; "OTP before account active".
- Uniqueness/one-time: "offer accepted once"; "email registered once".
- Ownership/identity: "a token authorizes only its own resource/user".
- Role/plan: "only Owners manage secrets"; "view-access role is Pro-only".
- Balance: "a credit reflects only settled funds".

## Step 3 - Attack the money fields
- Intercept the money-carrying request. Try, one at a time: tamper `price`/`amount`/`tax`/`shipping`/`discount`; `quantity` negative, decimal, zero, and huge (overflow); swap `currency` while leaving `amount`.
- If the server rejects mismatched totals, it is checking internal consistency only: keep the totals self-consistent and tamper the line items (note 01, Upserve).
- Test the inputs that are not prices but feed the total: country, region, shipping method, tax flags (note 06, Zyad).
- Full catalogue in `manipulation-and-bypasses.md`.

## Step 4 - Attack entitlement and identity
- For every feature the UI locks behind a role or plan, replay the underlying call with the locked/privileged value (note 14, note 25).
- Flip entitlement booleans (or blank a "already used" array) in the **response** and re-perform the action; it only counts if a backend endpoint accepts the follow-up (note 13, note 16).
- Build a role-by-endpoint matrix: capture the highest role's traffic, replay each call as each lower role and unauthenticated.

## Step 5 - Attack one-time and limited actions
- Plain replay first: re-send accept/redeem/claim/bonus/referral N times (note 11 got 30 discounts on plain replay).
- Then race it: parallel requests, or an HTTP/2 single-packet / gated burst, to beat a non-atomic check-then-act (note 11, note 12).
- After any fix lands, retest with concurrency. "Check then act" patches frequently stay racy (note 11's first fix did).

## Step 6 - Attack the workflow (forced browsing)
- Skip steps: call a later or completion endpoint directly without the prerequisite (note 23, note 12).
- Repeat and reorder steps; go back; submit a step twice.
- Delete a mandatory parameter entirely, the key not just the value, to hit an unvalidated branch (note 19, hover.com).
- Read error messages from half-initialized state; look for states that never expire.

## Step 7 - Attack token binding
- Obtain a valid token/OTP for an object you own, then replay the verify step with another object's id (note 20, Eternal).
- Check every token is bound to the user, resource, and action it should authorize, and that verify re-checks all three.

## Step 8 - Apply the impact filter, then prove net benefit
- CIA filter: does the result break confidentiality, integrity, or availability? Editing only your own data, or a wrong non-financial number on your own account, usually is not reportable (note 08).
- Prove the attacker is actually better off, end to end: paid less and the order is fulfilled, got the entitlement and used it, claimed the resource. A changed field or a lower displayed number is not enough (notes 04, 18).
- Verify through checkout/settlement or a test environment with test cards if production would charge real money (note 15).
- Capture minimal evidence and stop. Do not drain balances, claim real third-party resources, or enumerate beyond proof.

## Pitfalls seen in the sources
- A lower number on the payment page is meaningless if settlement re-validates it (note 04).
- A displayed price change can be dismissed as front-end only; confirm server-side effect (note 15).
- Triage may fail to reproduce flows that redirect (mobile "see retailers" opened a website): provide a video PoC (note 16).
- Obvious amount tampering is heavily duplicated; lead with a non-obvious vector or a bigger impact (note 10).
