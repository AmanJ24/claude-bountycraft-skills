# Race condition reporting

Races are disputed when the report looks like a one-off fluke. A good report proves the behavior is caused by concurrency (sequential baseline vs parallel result), quantifies the broken invariant, and names the non-atomic gap and its fix. Template and lessons below.

## Write-up template
1. **Title**: the invariant broken by concurrency. "Faucet per-user cap bypassed via concurrent claims"; "Email-change race yields a verified email I do not own".
2. **Summary**: one or two sentences, lead with the demonstrated impact (30 tokens from an 11-token cap; a single retest paid out multiple times; a verified email for a domain I do not control).
3. **The non-atomic gap**: name exactly where check and act are separated. "The balance check and the debit are separate statements with no transaction or row lock"; "the confirmation email's recipient and its token are read from different sources, so two parallel changes cross them."
4. **Baseline vs parallel (the proof it is a race)**: show the sequential result, then the parallel result. "Sequentially, 6 invites give 6 rejections and 1 email. Sent in parallel (single-packet attack, 6 requests), they produced 2 valid emails." This contrast is what distinguishes a race from normal behavior.
5. **Steps to reproduce**: the exact requests, the delivery technique (single-packet attack vs last-byte sync), how many requests, and how many attempts it took. Include the Turbo Intruder script (`Engine.BURP2`, `concurrentConnections=1`, `gate`/`openGate`) or the Burp Repeater group ("Send group in parallel"). On test data only.
6. **Impact and scale**: quantify (tokens, dollars, counts, accounts); state whether it is repeatable and the top primitive it reaches (verified-email forgery -> invitation hijack -> account takeover).
7. **Remediation**: the atomic fix. Atomic conditional update (`UPDATE ... WHERE balance >= price`), pessimistic lock (`SELECT ... FOR UPDATE`), optimistic locking with a version column, a uniqueness constraint as defense in depth, an idempotency key, a serialized critical section, consistent single-step session/ORM writes, and never using one storage layer (session) to protect another (DB).

## Lessons from this set
- **Benchmark sequentially first, and include it.** The whole case rests on "sequential does X, parallel does Y" (note 01). A parallel-only screenshot is not proof.
- **Quantify the overrun.** "30 tokens from an 11-token cap" and "$200 from a $100 credit" are concrete; "it duplicated" is not (notes 11, 12).
- **Tie duplicates to real impact.** Duplicate follows or a $0 extra credit is informational until you show over-count, double-spend, or a stuck/undeletable state (notes 13, 14).
- **State the delivery technique and reliability.** Say single-packet vs last-byte sync, request count, and attempts; triagers reproduce faster and rate higher when the method is explicit (notes 03, 06).
- **Retest after the fix; patches are often incomplete.** The flag-submission fix was partial; Stripe's first fix still raced. Re-run with concurrency and report if it still wins (notes 17, 11; business-logic note 11).
- **Escalate single-endpoint races beyond the immediate bug.** A verified-email-forgery race was only interesting because it chained to invitation hijack and OpenID account takeover; look for the downstream primitive (note 01).

## Redaction and safety
- Use test accounts and test data; never actually drain a balance, over-claim real value, or corrupt a real user's state beyond a minimal, reversible proof.
- Redact tokens, signed URLs, session cookies, and any real victim data.
- State what you did not do (did not withdraw, did not enumerate further); stopping at proof raises trust and severity.
