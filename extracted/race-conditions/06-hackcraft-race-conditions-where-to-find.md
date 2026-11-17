# Hackcraft: where to find race conditions, Turbo Intruder engine selection, and lab walkthroughs

- **Vuln type / sub-variant:** methodology/technique (practical assessment guidance plus Turbo Intruder configuration nuances; limit-overrun and multi-endpoint lab runs)
- **Target type:** Apps with coupons, tiered resource limits (free-tier object counts), payment then fulfilment flows, multi-step object creation, email activation, invitation acceptance.
- **Recon signal:**
  - Scanners cannot detect these, so they are skipped unless a coupon feature is present. Treat any multi-step operation, not only validation checks, as a candidate.
  - Check-then-act code paths: coupon redemption, paywall limits on projects, dashboards or workspaces, a window between payment confirmation and order finalization, objects built across several DB operations.
  - Real examples cited: HackerOne #429026 (multiple payouts for one retest via concurrent retest confirmations), HackerOne #1913309 (Firefox Monitor email-address limit exceeded), CVE-2022-4037 (GitLab email forgery).
- **Root cause:** Timing gaps between functions that each behave correctly in isolation; the check and the action are not atomic; cart or object state stays mutable after validation; partially built objects have uninitialized security fields.
- **Exploitation steps:**
  1. Test protocol support directly against the target, not through Burp (the proxy may downgrade to HTTP/1.1): `curl -s --http2 -o /dev/null -w '%{http_version}' https://target`. Output 2 means single-packet is available; 1.1 means fall back to last-byte sync.
  2. Engine choice in Turbo Intruder: Engine.BURP2 is the default and recommended. It negotiates via ALPN and uses single-packet over HTTP/2, falling back to last-byte sync over HTTP/1.1. If you specifically want last-byte sync on a server that also speaks HTTP/2, use Engine.THREADED, because BURP2 would multiplex everything on one connection and defeat the one-connection-per-request requirement. Burp Repeater "Send group in parallel" uses the same mechanics.
  3. Limit-overrun lab: strip unnecessary headers to keep each HTTP/2 frame small, queue about 20 coupon requests, release with the single-packet attack. Discount stacked to bring the total from 1,069.60 down to 15.40. With last-byte sync the window was too narrow to solve the lab.
  4. Multi-endpoint lab: apply the three-question collision test (where is state stored, edit vs append, what is it keyed on; here the session cookie keys both add-to-cart and checkout). Put an affordable item in the cart, then in one Turbo Intruder burst send a checkout request plus two add-to-cart requests for the expensive item. Order of queueing does not matter since all are released together; the second add-to-cart gives another chance to land in the validation-to-completion window. Result: order completed with the expensive item and the balance ended up negative.
  5. Turbo Intruder details for multi-endpoint scripts: set endpoint and host manually when queueing distinct requests, and set `requestsPerConnection` to the number of gated requests (3 in the example).
  6. Fit more requests per packet by dropping non-essential headers (Accept, Accept-Language, Accept-Encoding, User-Agent). Each HTTP/2 frame has a 9-byte header plus compressed headers; with lean requests 20-30 per packet is realistic. Content-Length need not be set manually.
  7. Connection warming note: the first request on a new or idle back-end connection is slower because of connection setup, not endpoint cost. Benchmark by grouping requests and prepending a GET in a single connection. The author found Repeater's sequential grouping too inconsistent to benchmark in the lab, and went straight to Turbo Intruder single-packet.
- **Bypasses used:** single-packet attack, last-byte sync (Engine.THREADED), connection warming, header stripping to fit more requests per packet, duplicate attack requests to raise odds.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Add race testing to the checklist for every state-changing multi-step feature, not just coupons: tier limits, payments, registrations, activation and invitation flows. Verify HTTP version out of band, choose the engine to match the technique, and keep requests lean. Remediation list: atomic conditional UPDATE (balance >= price), pessimistic locking (SELECT ... FOR UPDATE), optimistic locking with a version, idempotency keys, serialized critical sections, consistent session/ORM writes, no cross-layer protection.
- **Source:**
  - Title: Race Conditions and Where to Find Them
  - Author: Themis Zoumpoulakis (Hackcraft)
  - URL: https://www.hackcraft.gr/2026/04/race-conditions-and-where-to-find-them/
  - Date: 2026-04-07
  - Disclosed report: none (references HackerOne #429026 and #1913309 and CVE-2022-4037)
  - Raw file: `sources/race-conditions/06-hackcraft-race-conditions-where-to-find.md`

Tags: methodology, reference, limit-overrun, multi-endpoint, partial-construction, check-then-act, toctou, single-packet-attack, last-byte-sync, turbo-intruder, connection-warming, send-group-parallel, benchmarking, predict-probe-prove, defense, financial-loss
