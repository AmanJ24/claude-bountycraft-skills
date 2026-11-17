---
title: "Chapter 22: Race Conditions (Bug Bounty Handbook)"
author: mirshad0w
url: https://raw.githubusercontent.com/mirshad0w/bug-bounty-handbook/main/22-race-conditions.md
date_published: unknown
date_scraped: 2026-11-17
platform: github
disclosed_report: none
bounty: not stated
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

# Chapter 22 - Race Conditions

> Most apps assume requests arrive one at a time. Fire many at once and that assumption shatters, enabling double-spends, limit bypasses, and logic breaks.
> Learn the window first, then the tooling that lands inside it.

---

## Learning Objectives

By the end of this chapter you will be able to:

- Explain race conditions and the TOCTOU (time-of-check to time-of-use) problem at the level of database rows, locks, and request lifecycles.
- Identify features whose "check then act" logic is vulnerable to concurrent-request attacks.
- Exploit races using Burp Repeater's parallel send, the HTTP/2 single-packet attack, and Turbo Intruder.
- Recognize limit-overrun, multi-use, state-machine, and multi-endpoint race patterns.
- Reason about network jitter and how to neutralize it so the server-side window is what decides the outcome.
- Report races with reliable, reproducible proof and a correct concurrency-control fix.

---

## Visual Overview / Mind Map

```
   Single request (safe):
     READ balance=100 --> CHECK >=100 --> WRITE balance=0 --> withdraw 100
                                                            (next request sees 0)

   Two concurrent requests hit the SAME window:
     R1: READ 100 --> CHECK ok --.
     R2: READ 100 --> CHECK ok --|--> both passed the check on stale state
     R1: WRITE 0  --> withdraw 100
     R2: WRITE 0  --> withdraw 100   <-- 200 withdrawn from a 100 balance

   The window = the gap between READ/CHECK and WRITE/COMMIT.
     Tighten your requests' arrival to fit inside it:
        Repeater "send group in parallel"  -> good
        HTTP/2 single-packet attack         -> ~same millisecond (best)
        Turbo Intruder                      -> scripted, high concurrency

   Attack surface (look for "you may only do this once/N times"):
     one-time    -> coupon, gift card, referral, vote, invite accept
     balance     -> withdraw, transfer, purchase, points spend
     state       -> confirm/cancel, approve, email/role change
     multi-step  -> add-to-cart + apply-discount + checkout interleave
```

---

## Why This Chapter Exists

Race conditions exploit the gap between checking a condition and acting on it. When two requests interleave, an app can apply a one-time coupon twice, let a balance go negative, or push an object through two conflicting state transitions at once. For years these bugs were dismissed as "theoretical" because you could rarely land two requests close enough together to matter over a jittery network. Modern tooling, chiefly Burp's HTTP/2 **single-packet attack**, collapsed that timing problem, and races moved from folklore to a reliably reproducible, well-paid bug class.

It follows business logic (Ch. 21) because a race is often the *mechanism* by which you break a logical limit: the limit is enforced correctly for one request at a time, and the flaw is only visible under concurrency.

---

## Why This Knowledge Matters

- **High impact in financial and limit features**: double-spend, over-withdrawal, coupon/credit multiplication, quota bypass.
- **Underexplored** by many hunters who never got comfortable with the tooling, so ROI stays high.
- **Deepens your model of state, concurrency, and atomicity**: the same reasoning explains database isolation levels, idempotency keys, and distributed-system bugs.
- **Composes with everything**: a race can turn a "harmless" endpoint into an account-takeover or privilege-escalation primitive when combined with other logic.

---

## Prerequisites

- Chapters 00-21 (**M0-M21**). Business logic (Ch. 21), Burp Repeater/Intruder (Ch. 07), HTTP/2 basics (Ch. 01), and enough scripting to read Turbo Intruder Python (Ch. 04).

---

## Deep Theory: The Window, TOCTOU, and Why Networks Hide It

### TOCTOU: check here, use there

Almost every sensitive operation is really two steps separated in time:

```
time-of-CHECK :  is this coupon unused?  is balance >= amount?  is status == 'pending'?
        ...gap...
time-of-USE   :  mark coupon used     ;  balance -= amount   ;  set status = 'approved'
```

If the state that was true at the **check** can be changed by another request before the **use** commits, the check is worthless. This is the *time-of-check to time-of-use* (TOCTOU) problem. The vulnerable window is the wall-clock gap between reading/validating state and durably writing the new state. Databases can make each statement atomic, but the *application* rarely wraps the whole check-then-act sequence in a single atomic transaction or a row lock, so two requests both read the old value, both pass the check, and both write.

### Where the window actually lives

The window is not "in the network." It is server-side: the code path between the SELECT that validates and the UPDATE/COMMIT that persists. That path may include ORM hydration, business rules, external calls, and only then the write. The more work sits between check and commit, the wider the window, which is why some races are trivial to hit and others need the tightest possible request alignment.

### Why the network used to hide the bug

To exploit the window you must get two requests to be *inside it simultaneously* on the server. Over the internet, requests you send back-to-back arrive with **jitter**: variable per-request latency of several to tens of milliseconds. If the window is 1-2 ms, jitter of 20 ms means your requests almost never overlap, and the bug looks unexploitable. The whole art of modern race exploitation is **removing jitter** so the server's own window decides the outcome, not the network.

### The single-packet attack (why HTTP/2 changed everything)

James Kettle's single-packet attack exploits HTTP/2 multiplexing: you can place many requests in **one TCP packet**, so they arrive at the server at effectively the same instant, eliminating most network jitter. Over HTTP/1.1 the equivalent trick is the **last-byte-sync** technique: send all but the final byte of each request in advance, then release the final bytes together. Both aim at the same goal: make the requests' arrival spread far smaller than the server-side window.

```
HTTP/1.1 last-byte sync         HTTP/2 single-packet
 req1: [-------------X]          [ req1 | req2 | ... | reqN ]  <- one packet
 req2: [-------------X]                     |
 ...   withhold final byte,                 v
 then release all X together      server parses ~simultaneously
```

---

## Topics

Tagged: **[CORE]** / **[USEFUL]** / **[SKIM]**.

### 1. The Concept: TOCTOU [CORE]

The app **checks** something (balance, coupon unused, stock, status) then **uses** it, but between those steps, another request sneaks in and passes the same stale check. Concurrency breaks the assumption of atomic, sequential processing that the code was written under.

### 2. Where Races Hide [CORE]

- **One-time actions:** coupons, gift cards, referral bonuses, vote/like once, invite acceptance, one-per-account signup gifts.
- **Balance/quota operations:** withdrawals, transfers, purchases, points/credits spend, API rate limits, seat counts.
- **State transitions:** applying/redeeming/confirming, cancel-vs-refund, account creation, email/role change, order status.

### 3. Exploitation Techniques [CORE]

- **Burp Repeater "send group in parallel"**: the fastest way to try a race by hand; Burp automatically uses the single-packet attack over HTTP/2.
- **HTTP/2 single-packet attack**: many requests in one packet for ~single-millisecond arrival.
- **HTTP/1.1 last-byte synchronization**: the fallback when the target is HTTP/1-only.
- **Turbo Intruder**: scripted, high-concurrency races and the `raceSinglePacket` gate for repeatable timing.

### 4. Race Patterns [USEFUL]

- **Limit overrun:** redeem a single-use item N times.
- **Balance manipulation:** withdraw/transfer more than you have (money, points, stock).
- **Multi-use of one-time tokens** (password-reset, OTP, invite).
- **State confusion:** two conflicting transitions at once (confirm + cancel).
- **Multi-endpoint (single-request) races:** interleave *different* endpoints that share state (e.g. apply-discount while checkout reads the cart).

### 5. Reliability & Proof [USEFUL]

Races are probabilistic. Run multiple attempts, capture the successful interleaving (the duplicated side effect), and show before/after state clearly. Reliability improves with tighter request alignment and, sometimes, with a **connection-warming** request that gets TLS/HTTP state established before the timed burst.

---

## Step-by-step Methodology

```
1. FIND     Locate "you may only do this once / up to N" logic:
            coupons, withdrawals, transfers, one-per-account rewards,
            state flips (confirm/cancel), OTP/reset consumption.
2. BASELINE Do the action once, normally. Note the exact request and the
            single-request success/refusal response. This is your control.
3. GROUP    Put 10-50 identical requests in a Repeater tab group (or a
            Turbo Intruder script). Fresh coupon/token if it's single-use.
4. FIRE     "Send group in parallel" (HTTP/2 single-packet). For HTTP/1
            targets, rely on last-byte sync (Burp/Turbo do this for you).
5. OBSERVE  Compare against baseline: >1 success? balance below zero?
            two state transitions applied? duplicate side effects?
6. RETRY    Nothing? Re-fetch state, widen the group, add a warming
            request, retry several times - races are probabilistic.
7. PROVE    Capture before/after state (balance, redemption count, DB-
            visible artifacts). Minimal, reproducible steps. Report.
```

---

## Code Samples

Illustrative, benign PoCs for authorised testing/labs. Only ever race *your own* test-account resources.

### Turbo Intruder: single-packet limit-overrun

```python
# Turbo Intruder script: fire 30 identical redeem requests in one packet.
# Extensions > Turbo Intruder > paste over a captured request, then queue.
def queueRequests(target, wordlists):
    # gate='race1' holds requests until the batch is released together.
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2)  # HTTP/2 single-packet
    for i in range(30):
        engine.queue(target.req, gate='race1')
    engine.openGate('race1')  # release all ~simultaneously

def handleResponse(req, interesting):
    # Log status + a marker so you can count how many "succeeded".
    table.add(req)
```

### Turbo Intruder: HTTP/1.1 last-byte fallback

```python
# When the target is HTTP/1.1 only, use the classic last-byte sync engine.
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           engine=Engine.THREADED,
                           pipeline=False)
    for i in range(30):
        engine.queue(target.req, gate='g1')
    engine.openGate('g1')

def handleResponse(req, interesting):
    table.add(req)
```

### curl: quick-and-dirty parallel burst (weak, but zero-setup)

```bash
# Fire 20 concurrent redemptions. Network jitter makes this UNRELIABLE for
# tight windows - use it only to smoke-test wide windows; prefer Burp/Turbo.
URL='https://target.example/api/coupon/redeem'
TOKEN='session=REPLACE_WITH_YOUR_TEST_SESSION'
for i in $(seq 1 20); do
  curl -s -o /dev/null -w '%{http_code}\n' \
    -H "Cookie: $TOKEN" -H 'Content-Type: application/json' \
    --data '{"code":"WELCOME10"}' "$URL" &
done
wait   # any two 200s for a single-use coupon = a race
```

### Python: count successes across a burst

```python
# Concurrent burst with a thread pool; count how many "applied" responses
# come back for an action that should only ever succeed once.
import concurrent.futures, requests
URL = "https://target.example/api/coupon/redeem"
CK  = {"session": "REPLACE_WITH_YOUR_TEST_SESSION"}
def hit(_):
    r = requests.post(URL, cookies=CK, json={"code": "WELCOME10"})
    return r.status_code, ("applied" in r.text.lower())
with concurrent.futures.ThreadPoolExecutor(max_workers=30) as ex:
    res = list(ex.map(hit, range(30)))
print("applied:", sum(1 for _, ok in res if ok))  # >1 = race won
```

---

## Burp Suite Workflow

```
1. Proxy: capture the single sensitive request (redeem/withdraw/confirm).
2. Right-click > "Add to group" (or Ctrl-click several Repeater tabs).
   Duplicate the tab until the group holds 10-30 identical requests.
3. Refresh any single-use precondition (new coupon/token) so all requests
   in the group target a still-valid resource.
4. Group send drop-down > "Send group in parallel". Over HTTP/2 Burp uses
   the single-packet attack automatically; over HTTP/1 it uses last-byte
   sync. Watch the response times cluster tightly.
5. Sort responses by status/length; look for more successes than the logic
   should allow, or a balance/state that went impossible.
6. For high concurrency or scripting, send the request to Turbo Intruder.
```

Burp's **grouped Repeater send** is the single most important skill here; **Turbo Intruder** is for scale and for scripting the gate. Use **Logger** to keep a durable record of the winning interleaving for your report.

---

## Detection: Manual + Automation

### Manual

- **Read the logic for the word "once."** Any "you can only redeem/claim/transfer this once (or N times)" is a candidate. So is any state flip that should be one-way.
- **Establish the single-request baseline first.** You can only recognize a race win by contrast with the normal single-request refusal.
- **Diff behavior under load** by hand: one request refuses, a parallel group of ten does not. That gap is the finding.

### Automation

- **Turbo Intruder** scripts the burst and records per-response outcomes so you can count successes programmatically.
- **Custom thread-pool scripts** (Python) are fine for wide windows and CI-style regression checks.
- Beware naive scanners: most **cannot** land requests tightly enough to trigger a real race, so absence of a scanner hit means nothing here. This class is manual-first.

---

## Modern Protections & Bypass

- **Atomic database operations**: a single `UPDATE ... WHERE balance >= amount` (conditional update) or `INSERT` with a unique constraint makes check-and-act one indivisible step. **Bypass reality:** only closes the window if the check truly lives *in* the atomic statement; an app that SELECTs, validates in code, then UPDATEs is still racy.
- **Row/record locking** (`SELECT ... FOR UPDATE`, pessimistic locks) serializes concurrent access to the same row. **Bypass:** locks scoped to the wrong row, or a multi-endpoint race that touches different rows that jointly gate one limit.
- **Idempotency keys**: the client sends a unique key; the server processes each key once. Strong against replay-style duplication, but only if the key is required and truly deduplicated server-side.
- **Unique constraints / optimistic concurrency (version columns)**: reject the second writer. Effective when present on the right column.
- **Higher isolation levels** (serializable): the database aborts conflicting transactions. Costly, so often not used where it matters.

The recurring bypass theme: defenses that guard *one* row or *one* endpoint can be sidestepped by racing a **different** shared-state path that reaches the same limit.

---

## Real Disclosed Reports (conceptual summaries)

Read the public write-ups directly; these are conceptual pointers, not report text.

- **Single-packet "Smashing the state machine" (PortSwigger research, James Kettle).** Established the single-packet attack and showed multi-endpoint races that bypass limits by interleaving *different* endpoints sharing state. Lesson: the window can span endpoints, not just repeated calls. -> `https://portswigger.net/research/smashing-the-state-machine`
- **Coupon/gift-card multiplication (many programs).** Redeeming a single-use code concurrently credited the account multiple times, giving direct financial impact. Lesson: any "once per account" credit is a prime target.
- **Withdrawal/transfer over-spend (fintech programs).** Concurrent withdrawals each read the pre-debit balance and all succeeded, taking the balance negative. Lesson: balance checks outside an atomic update are classic TOCTOU. -> PortSwigger Academy race-condition labs and public HackerOne reports: `https://portswigger.net/web-security/race-conditions`

---

## Common Mistakes

- **Sending requests too slowly.** Manual one-at-a-time sends miss the window; use grouped parallel send / single-packet.
- **Giving up after one attempt.** Races are probabilistic: widen the group and retry several times.
- **Reusing an already-consumed resource.** If the coupon/token was spent on your baseline, refresh it before the burst.
- **Testing over HTTP/1 without last-byte sync** and concluding "not vulnerable": the tool, not the app, was the limiting factor.
- **Weak proof.** Capture clear before/after state so triagers can see the duplicated effect.

---

## Advanced / Hunter Notes

- **Multi-endpoint races beat single-row locks.** If endpoint A and endpoint B both decrement one shared allowance, interleaving A and B can pass a lock that only guards A. This is the highest-value modern variant.
- **Connection warming.** A throwaway request just before the burst establishes TLS/HTTP/2 state, shrinking arrival spread and boosting win rate.
- **Widen the window on purpose.** A slow/large upload, a heavy `Referer`, or a cache-miss path in the same request can lengthen server-side processing, making a tight window easier to hit.
- **State-machine skips.** Some races don't duplicate an action; they push an object into an *impossible* state (both "cancelled" and "shipped"), unlocking downstream logic bugs.
- **Rate limits are races too.** "Max 5 OTP attempts" enforced with a read-modify-write counter can be overrun to brute past the limit.
- **Second-order races.** The duplicate effect may surface later (a nightly batch, a webhook) rather than in the immediate response; check downstream artifacts.

---

## Practical Work / Labs

1. **All PortSwigger Race Condition labs**, learning the single-packet attack end to end, including the multi-endpoint lab.
2. **Turbo Intruder drill:** script a `gate`-released burst against a limit feature and count successes programmatically.
3. **Detection drill:** in a practice app, find three "once/N-times" features, reason about each TOCTOU window, and predict which is widest.
4. **Fallback drill:** force an HTTP/1.1 target and reproduce a race using last-byte synchronization.

---

## Mini Challenges

- Redeem a one-time coupon multiple times via a parallel send and count the extra credits.
- Take a balance negative through concurrent withdrawals and capture before/after.
- Win a **multi-endpoint** race where two different endpoints share one allowance.

---

## Milestone

**M22 - Race Conditions.** You can find and reliably exploit race conditions using modern single-packet/parallel techniques, including multi-endpoint variants, and prove them cleanly.

---

## Self-Assessment: "I can..."

- [ ] I can explain TOCTOU and locate the server-side window between check and commit.
- [ ] I can spot "once/N-times" features likely vulnerable to races.
- [ ] I can use Burp's grouped parallel send and the single-packet attack.
- [ ] I can script a gated burst in Turbo Intruder and fall back to last-byte sync on HTTP/1.
- [ ] I can recognize limit-overrun, multi-use, state, and multi-endpoint races.
- [ ] I can produce reliable, reproducible before/after proof.

If most boxes are ticked, mark **M22** and continue.

---

## Key Takeaways

- A race is a TOCTOU bug: two requests pass the same stale check before either commits.
- The window is server-side; exploitation is about removing network jitter so requests land inside it.
- The single-packet attack (HTTP/2) and last-byte sync (HTTP/1) are the enabling techniques; Turbo Intruder scales them.
- Multi-endpoint races defeat naive single-row locks and are the highest-value modern variant.
- The durable fix is atomicity (conditional updates, locks, unique constraints, or idempotency keys), not "check more carefully."

---

## Bug Hunter Checklist

- [ ] Identified a "once / up to N" or one-way state feature.
- [ ] Captured the single-request baseline (success and refusal).
- [ ] Built a 10-30 request group with a still-valid single-use precondition.
- [ ] Sent group in parallel (single-packet) / last-byte sync on HTTP/1.
- [ ] Retried with a wider group and a warming request when the first burst missed.
- [ ] Tested a multi-endpoint variant sharing the same allowance.
- [ ] Captured before/after state as reproducible proof.

---

## Practice Exercises

1. Reproduce a coupon-multiplication race in a lab and report it with a counted before/after.
2. Script a Turbo Intruder gate burst and tabulate how many requests "won."
3. Explain, in writing, why a `SELECT`-then-`UPDATE` balance check is racy and how a conditional `UPDATE ... WHERE balance >= amount` fixes it.

---

## Chapter Summary

Race conditions are TOCTOU bugs: an application checks a condition (coupon unused, balance sufficient, status pending) and then acts on it, but the state can change in the gap between the two steps. When several requests enter that server-side window simultaneously, they all pass the same stale check and all commit: redeeming a one-time coupon many times, taking a balance negative, or forcing an object into two conflicting states. For years the network's jitter hid the bug; the HTTP/2 single-packet attack and HTTP/1 last-byte synchronization removed that jitter, making races reliably reproducible. Exploitation is a tight loop: baseline the single request, group many identical requests, fire them in parallel, and compare against the baseline, retrying because the outcome is probabilistic. The most powerful modern variant races *different* endpoints that share one allowance, defeating locks that guard only a single row. The real fix is atomicity (conditional updates, row locks, unique constraints, or idempotency keys) because "checking more carefully" never closes a window that concurrency can slip through.

---

## Final Chapter Checklist

- [ ] I can explain TOCTOU and where the window lives server-side.
- [ ] I can exploit single-endpoint and multi-endpoint races with the right tooling.
- [ ] I can neutralize network jitter (single-packet / last-byte sync).
- [ ] I can prove a race with clean before/after state.
- [ ] I can recommend the correct atomic/locking/idempotency fix.

---

## Further Reading (official / primary sources)

- **PortSwigger Academy: Race Conditions.** -> `https://portswigger.net/web-security/race-conditions`
- **PortSwigger research: "Smashing the state machine" (James Kettle).** -> `https://portswigger.net/research/smashing-the-state-machine`
- **Turbo Intruder (Burp extension) documentation.** -> `https://github.com/PortSwigger/turbo-intruder`
- **OWASP: Testing for Race Conditions.** -> `https://owasp.org/www-project-web-security-testing-guide/`

---

## Next Chapter Preview

**Chapter 23: HTTP Request Smuggling.**

Next we exploit disagreements between front-end and back-end servers about where one request ends and the next begins. This advanced class reaches back to your HTTP and proxy knowledge (Ch. 01/05) and can poison other users' requests.

> Finish M22 first. Then turn the page.

Capture status: FULL.
