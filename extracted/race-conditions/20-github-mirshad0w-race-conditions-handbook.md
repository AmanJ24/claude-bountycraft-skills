# Bug bounty handbook chapter: race condition checklist and common mistakes

- **Vuln type / sub-variant:** Methodology / taxonomy (reference chapter covering limit overrun, multi-use tokens, state confusion, multi-endpoint races, rate-limit races).
- **Target type:** Generic web apps, with focus on one-time actions (coupons, gift cards, referral, invites), balance operations (withdraw, transfer, points), and state transitions (confirm/cancel, email or role change).
- **Recon signal:** Any logic that says "only once" or "up to N", any one-way state flip, OTP or reset-token consumption, attempt-limited counters.
- **Root cause:** Check-then-act (TOCTOU) with the application not wrapping validation and write in one atomic statement or lock. The window is server-side, between the validating read and the committing write. Network jitter used to mask it until the single-packet attack removed that obstacle.
- **Exploitation steps:**
  1. Find a once/N-times feature or one-way state change.
  2. Baseline: perform the action once and record the success and refusal responses as the control.
  3. Group 10-50 identical requests (Repeater tab group or Turbo Intruder), using a fresh single-use precondition.
  4. Fire in parallel: HTTP/2 single-packet attack, or last-byte sync for HTTP/1.1.
  5. Compare to baseline: more than one success, negative balance, two conflicting states, duplicate side effects.
  6. Retry with a wider group and a connection-warming request, since races are probabilistic.
  7. Capture before/after state as proof, and check second-order effects (batch jobs, webhooks).
- **Bypasses used:** Defence-side notes: atomic conditional UPDATE, row locks, idempotency keys, unique constraints or version columns, serializable isolation. Recurring bypass theme: guards on one row or endpoint can be sidestepped by racing a different endpoint that touches the same shared allowance (multi-endpoint). Attacker tricks: connection warming, deliberately widening the window with slow or heavy request paths, rate limits (OTP attempt counters) overrun by read-modify-write races, pushing objects into impossible states.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Testable checklist: identify once/N feature, capture baseline, 10-30 request group with valid precondition, parallel send, retry wider with warming, try a multi-endpoint variant, record before/after proof. Common mistakes: sending too few requests or too slowly, giving up after one attempt, reusing an already-consumed coupon or token, testing over HTTP/1 without last-byte sync and concluding not vulnerable, not using single-packet, not benchmarking against a single-request baseline, weak proof. Naive scanners rarely land a real race, so this class is manual-first. Curl or thread-pool bursts only suit wide windows. Derivative secondary source: its disclosed-report section is conceptual pointers to Kettle's "Smashing the state machine", not specific reports.
- **Source:**
  - Title: Chapter 22: Race Conditions (Bug Bounty Handbook)
  - Author: mirshad0w
  - URL: https://raw.githubusercontent.com/mirshad0w/bug-bounty-handbook/main/22-race-conditions.md
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/race-conditions/20-github-mirshad0w-race-conditions-handbook.md`

Tags: methodology, taxonomy, reference, toctou, check-then-act, non-atomic, single-packet-attack, last-byte-sync, turbo-intruder, connection-warming, multi-endpoint, limit-overrun, rate-limit-bypass, otp-brute, sub-state, benchmarking, defense
