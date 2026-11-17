# YesWeHack bug bounty guide to race conditions: concepts, sync techniques, lab walkthrough, mitigations

- **Vuln type / sub-variant:** methodology/technique (introductory guide; limit-overrun walkthrough on the PortSwigger coupon lab)
- **Target type:** Generic web apps with shared state; the worked example is an e-shop coupon endpoint.
- **Recon signal:** Scanners miss these bugs, so hunters need to understand the app's inner workings and look at checkout, coupon, payment and transfer logic. The lab signal is a discount that a normal request applies only once.
- **Root cause:** Threads or processes modify shared data without synchronization, so the result depends on timing. The guide illustrates this with a non-atomic counter increment that loses updates, and notes that web servers give each request its own thread, so DB updates and token generation can interleave.
- **Exploitation steps:**
  1. Confirm the server's HTTP version, then pick the sync technique: last-byte sync for HTTP/1.1 (hold the request open and finish several close together), single-packet for HTTP/2 (multiplex requests on one connection, hold back a fragment of each, pause briefly, then send the final fragments so they share one TCP packet).
  2. Lab walkthrough: add the expensive item to the cart, apply the coupon once (price still above balance), capture the apply-coupon request and send it to Turbo Intruder.
  3. Turbo Intruder template used: `RequestEngine(endpoint=target.endpoint, concurrentConnections=10, engine=Engine.BURP2)`. Comments in the template: use Engine.BURP2 for HTTP/2 single-packet; use Engine.THREADED or Engine.BURP for HTTP/1 only servers. Queue 20 copies with `engine.queue(target.req, gate='race1')`, then `engine.openGate('race1')`. `handleResponse` just adds results to the table. A negative timestamp means the server answered before the request was complete. (Note: PortSwigger's own topic page uses `concurrentConnections=1`.)
  4. Result: the discount stacked repeatedly, dropping the price to an affordable amount (about 37.62 against a 1,337 starting price in the lab).
  5. Expect probabilistic exploitation. Some races are deterministic with the right setup, but many need many attempts.
- **Bypasses used:** last-byte sync, single-packet attack, Turbo Intruder gating.
- **Impact:** n/a (reference). The guide lists generic impacts: data corruption, business logic abuse (coupons, transfers), denial of service via unstable state, and broken access control such as skipping a multi-step login.
- **Bounty:** n/a (reference)
- **Lesson / pattern:** The guide cites CVE-2022-4037 (GitLab email verification forgery), CVE-2024-58248 (nopCommerce, no locking on order placement allowing repeated gift card redemption) and Dirty COW as examples of the class. Mitigations it lists: mutex or lock, ACID transactions with SELECT ... FOR UPDATE, idempotency and optimistic locking, atomic validate-and-update, distributed locks, rate limiting, and race detectors and stress tests in CI. Useful as a quick checklist and a reading list (Doyensec database race conditions post, RyotaK's Flatt post, Amin Nasiri's single-packet deep dive).
- **Source:**
  - Title: Concurrency exploits: The ultimate Bug Bounty guide to exploiting race condition vulnerabilities in web applications
  - Author: Alex Brumen (YesWeHack)
  - URL: https://www.yeswehack.com/learn-bug-bounty/ultimate-guide-race-condition-vulnerabilities
  - Date: 2025-09-26
  - Disclosed report: none
  - Raw file: `sources/race-conditions/05-yeswehack-race-condition-guide.md`

Tags: methodology, reference, limit-overrun, single-packet-attack, last-byte-sync, turbo-intruder, non-atomic, missing-lock, unlimited-redemption, defense
