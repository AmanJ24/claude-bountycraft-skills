# Every.org staging: duplicate follows via parallel Follow requests

- **Vuln type / sub-variant:** Social-action uniqueness violation (limit-overrun on a once-only relationship)
- **Target type:** Social "follow a user" endpoint on a nonprofit platform (staging environment)
- **Recon signal:** Following twice normally returns an error, which implies a server-side uniqueness check that can race.
- **Root cause:** Follow was not idempotent. The existence check and insert were separate, so concurrent requests all saw "not following" and created duplicate records. The fix was making the operation idempotent.
- **Exploitation steps:**
  1. Open any user profile, intercept the Follow click in Burp.
  2. Send it to Turbo Intruder and add a dummy header containing the `%s` placeholder.
  3. Configure the engine with 30 concurrent connections, 100 requests per connection, pipelining off.
  4. Queue 30 requests on one gate, open the gate, wait for completion.
  5. Inspect status codes: multiple 201 Created responses mean the user was followed multiple times.
- **Bypasses used:** None. Gated release via Turbo Intruder over many connections.
- **Impact:** Repeated follows from one account (inflated counts, data integrity). Low severity.
- **Bounty:** Not stated (hidden on report).
- **Lesson / pattern:** Use a duplicate-vs-error contrast: if the serial behavior is a rejection, many 2xx responses in a parallel burst prove a race. Idempotent writes (unique constraints) are the fix.
- **Source:**
  - Title: Race Condition when following a user
  - Author: bugra (Bugra Eskici)
  - URL: https://hackerone.com/reports/927384
  - Date: 2020-07-19
  - Disclosed report: https://hackerone.com/reports/927384
  - Raw file: `sources/race-conditions/13-h1-927384-follow-user-race.md`

Tags: social-action, limit-overrun, check-then-act, turbo-intruder, send-group-parallel, state-corruption
