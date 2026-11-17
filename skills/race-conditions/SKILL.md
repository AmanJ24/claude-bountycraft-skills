---
name: race-conditions
description: Use when testing any feature where two or more requests touching the same record at the same time could collide: single-use or limited actions (redeem coupon/gift card, claim faucet/bonus, withdraw, vote, follow, invite), multi-step flows (payment vs add-to-cart, login vs MFA enforcement), email-change/confirmation, registration (partially-built objects), and rate-limited OTP/token checks. Covers recognizing check-then-act (TOCTOU) sub-states, the single-packet attack and last-byte-sync delivery, the predict/probe/prove methodology, multi-endpoint window alignment, false positives, and reporting. Research/methodology only: it does not send requests to live targets.
---

# Race Conditions

Synthesized from 19 real bug bounty findings and references (James Kettle's "Smashing the state machine" and the single-packet-attack papers, the PortSwigger Race Conditions labs/topic, RyotaK/Flatt's first-sequence-sync, the Turbo Intruder docs, HackerOne disclosed reports incl. a $5,000 faucet over-claim and a double-paid retest, practitioner guides). Every pattern traces to `sources.md`. Disclosed bounties here ran from low (social-action dupes) to $5,000 (a Critical faucet over-claim); the money is in limit-overrun on value and in races that corrupt auth/account state.

**The one idea:** a race condition is a **check-then-act gap**. Between the moment the server checks a condition (coupon unused, balance sufficient, email unconfirmed, user not yet a member) and the moment it acts on that check, there is a short-lived **sub-state**, usually about 1ms, that the request enters and leaves. If two requests both pass the check before either finishes acting, the invariant breaks. You find these by making requests **collide inside that window**, which the modern **single-packet attack** (HTTP/2) makes reliable even against remote servers. Everything in race-condition testing is multi-step: **predict the collision, probe for it, prove it**.

**Scope rule:** methodology and guidance only. It does not fire requests at live targets. Real testing needs an authorized, in-scope program and accounts you control; racing moves real money, grants real entitlements, and corrupts real state, so stay on test data and stop at proof (see `../../CLAUDE.md`).

**Tooling note:** reliable exploitation needs Burp Repeater's "Send group in parallel" or Turbo Intruder (the single-packet attack / last-byte sync). In-page `fetch` concurrency only approximates this and only hits wide race windows; it will not hit sub-millisecond windows. See `references/the-single-packet-attack.md`.

## 1. Recognize it

A race lives wherever the app **reads state, decides, then writes**, without making that sequence atomic. The input is a perfectly valid request; the bug is sending it (or a partner request) more than once at the same instant.

Signals, roughly in order of how often they paid out in this set:
- A **single-use or limited action** with value or security impact: redeem coupon/promo, claim a faucet/bonus/survey credit, withdraw, apply a discount, submit a flag for points, follow, invite, join (notes 10, 11, 12, 13, 15, 17). This is **limit overrun**, the simplest class.
- A **multi-step flow inside one request** with a security-relevant ordering: login sets the session before MFA is enforced; an order validates payment before it is finalized; an object is inserted before a security field is set (notes 03, 06).
- An **email-change / confirmation** flow, or any action where the record a token is bound to can be edited between "send" and "use" (single-endpoint collision; GitLab CVE-2022-4037, note 01).
- **Requests that all process sequentially despite being sent in parallel**: a sign of per-session locking (PHP native sessions), which you defeat with a distinct session per request (notes 01, 03).
- A **rate-limited OTP/token** check (login PIN, 2FA code) where winning the race means many attempts land before the limit applies (notes 03, 04).
- **Security tokens derived from a timestamp**: two requests on the same tick collide (time-sensitive, note 03).

## 2. Where it hides

Feature classes that actually produced findings. Full list with examples in `references/attack-surfaces.md`.

- **Coupons / promos / gift cards / discounts**: redeem the same code many times in parallel to stack it below cost (Instacart, note 10; the PortSwigger limit-overrun lab).
- **Faucets / balances / withdrawals / credits**: a per-user cap or balance checked then decremented non-atomically; concurrent claims exceed the cap or double-spend (Cosmos/Starport faucet $5k, note 11; Slack survey credit $100 -> $200, note 12).
- **Payment vs cart (multi-endpoint)**: race an add-to-cart into the window between payment validation and order completion (PortSwigger multi-endpoint lab; note 06).
- **Login vs MFA (sub-state)**: fire the login in parallel with a request to a sensitive authenticated page to land in the logged-in-but-MFA-not-yet-enforced window (note 03).
- **Email change / confirmation (single-endpoint)**: two parallel changes so the token mailed for one address validates another, yielding a verified email you do not own (GitLab, note 01).
- **Social / membership actions**: follow, invite, join, vote raced to duplicate or to corrupt state so a member becomes undeletable (Every.org follow note 13, Omise invite note 15, hacker101 group note 14).
- **Registration / object creation (partial construction)**: race an authenticated request against a half-built object whose security field is still uninitialized (note 03).
- **Rate-limited OTP / token**: win the race so hundreds of guesses land before the 5-attempt limit (notes 03, 04).

## 3. Test methodology

The predict/probe/prove loop from the Kettle whitepaper. Full step-by-step in `references/testing-methodology.md`.

1. **Confirm the delivery technique.** Check HTTP version out of band: `curl -s --http2 -o /dev/null -w '%{http_version}' https://target`. `2` means the single-packet attack is available; `1.1` means fall back to last-byte sync. See `references/the-single-packet-attack.md`.
2. **Predict collisions.** List objects with a security control (user, session, order, balance). For each, list endpoints that read-and-act or write it. Prune with **three questions**: (a) **where is the state stored?** client-side (an emailed JWT) cannot collide; server-side persistent is ideal; session state is partly protected. (b) **editing or appending?** appending is usually only limit-overrun; editing a record has real collision potential. (c) **what is the operation keyed on?** two operations must share the same key to collide (a reset keyed on userid cannot collide across two users; one keyed on sessionid can).
3. **Probe.** Benchmark first: send the group **sequentially** (separate connections, seconds apart) and record the baseline. Then send the **same group in parallel** (single-packet attack, or last-byte sync). Hunt for **any deviation**: different status/body, extra or misrouted emails, a changed session, a 500, a suspiciously fast or slow response. Use 20-30 requests during discovery to beat server-side jitter.
4. **Prove.** Reduce to the **minimum requests** (usually two) that still reproduce, because extra requests can overwrite a good end state. Then escalate: treat the race as a structural weakness and find the highest-impact primitive it gives.
5. **Align multi-endpoint windows** when two different endpoints must collide: benchmark to separate connection-setup delay (ignore it, first request only) from endpoint-processing delay; use **connection warming** (a dummy GET first) and, if needed, server-side delay via a dummy rate-limit flood (keeps single-packet viable) rather than a client-side delay (which splits packets).
6. **Defeat session locking** (requests serialize despite parallel send) with a **distinct session token per request**.
7. **Stop at proof** on test data: demonstrate the over-count / double-spend / forged verification once, do not drain balances or claim real resources.

## 4. The delivery technique (what makes it work)

Summary; full detail, tooling config, and the expert escalation in `references/the-single-packet-attack.md`.

- **Single-packet attack (HTTP/2, the default).** Open one connection, start 20-30 requests but withhold a tiny final fragment of each (an END_STREAM-bearing DATA frame), wait ~100ms, then release all final fragments together so the OS packs them into one TCP packet. This eliminates network jitter, making a remote race as reliable as a local one (benchmarked ~1ms spread vs 4ms for last-byte sync; one real bug fell in ~30s vs 2+ hours). Caps at ~20-30 requests (the ~1,500-byte packet limit).
- **Last-byte sync (HTTP/1.1 fallback).** Send all but the last byte of each request over parallel connections, then release the last bytes together. Burp Repeater and Turbo Intruder pick this automatically for HTTP/1.1.
- **Tooling.** Burp Repeater: group the tabs, "Send group in parallel". Turbo Intruder: `Engine.BURP2` + `concurrentConnections=1` for HTTP/2 single-packet, queue with a `gate`, release with `openGate` (`race-single-packet-attack.py`). Strip non-essential headers to fit more requests per packet. A negative timestamp in Turbo Intruder means the server answered before the request completed (also a static-file test).
- **First sequence sync (expert escalation, note 04).** When a limit-overrun or OTP-brute needs thousands of synchronized requests (beyond the ~20-30 cap), RyotaK's technique uses IP fragmentation + TCP sequence reordering to land ~10,000 requests in ~166ms. The real ceiling becomes the server's `SETTINGS_MAX_CONCURRENT_STREAMS` (Apache 100, Nginx 128, Go 250; some stacks effectively unlimited). Needs custom tooling (operates at OSI layers 3-4, so Burp cannot do it).

## 5. Common false positives

- **Client-side state cannot race.** If the security decision rides in a value the client holds (an emailed JWT reset token, a signed cookie), two requests cannot collide on server state. Confirm the state is server-side (note 01).
- **Append-only actions are usually just limit-overrun, not a deeper bug.** Adding another item/email rarely collides in an interesting way; editing a single record does. Match the claim to the sub-variant (note 01).
- **Operations keyed on different records never collide.** A reset keyed on each victim's own userid row cannot be crossed; you need two operations sharing a key (note 01).
- **Apparent success from jitter, not a race.** A one-off anomaly under parallel send may be noise. Benchmark sequentially first and reproduce the parallel result; a real race deviates consistently from the sequential baseline (note 03).
- **Idempotent or constrained endpoints.** If the action is idempotent or the DB has a uniqueness constraint, duplicates collapse and there is no impact (the follow bug was fixed by making it idempotent, note 13).
- **"It worked once" without impact.** Two duplicate follows or an extra survey credit of $0 value is informational; tie it to real over-count, double-spend, or state corruption (notes 13, 12).
- **In-harness non-reproduction.** Failing to win a tight race with in-page `fetch` is a tooling limit, not proof of safety; the single-packet attack is required (scope/tooling note).

## 6. Reporting

What made reports in this set land well (template in `references/reporting.md`):
- **Show the sequential baseline vs the parallel result.** "Sequentially, 6 invites give 6 rejections and 1 email; in parallel they gave 2 valid emails." The contrast is the proof the issue is concurrency, not normal behavior (note 01).
- **Lead with the broken invariant and its impact.** "The 11-token faucet cap was exceeded (30 tokens from 50 concurrent claims)"; "a single retest was paid out multiple times"; "I hold a verified email I do not own." Quantify (tokens, dollars, counts) (notes 11, 16, 01).
- **Name the exact non-atomic gap.** "Balance is checked and decremented in separate statements with no lock/transaction"; "the confirmation email's recipient and token are read from different sources." Give the fix: atomic conditional update, row lock (`SELECT ... FOR UPDATE`), uniqueness constraint, idempotency key.
- **State the delivery technique and reliability.** Single-packet vs last-byte sync, how many requests, how many attempts; include the Turbo Intruder script or the Repeater group.
- **Retest after the fix.** Race patches are frequently incomplete (the flag-submission and Stripe fixes needed a second round); re-run with concurrency and report if it still races (notes 17, 11).

## 7. Sources

See `sources.md` for the full list (title, author, URL, disclosed report, bounty) and `../../extracted/race-conditions/` for the per-writeup notes each claim is drawn from. Deeper references:
- `references/attack-surfaces.md`: feature classes with example reports
- `references/the-single-packet-attack.md`: the delivery technique, tooling config, last-byte sync, first-sequence-sync
- `references/testing-methodology.md`: predict/probe/prove, the three-question collision test, window alignment
- `references/sub-variants-and-cases.md`: the taxonomy with real examples
- `references/reporting.md`: write-up template
