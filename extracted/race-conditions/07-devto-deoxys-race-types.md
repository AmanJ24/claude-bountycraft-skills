# DEV Community overview of race condition types, with multi-endpoint sync workarounds

- **Vuln type / sub-variant:** methodology/taxonomy (beginner-level restatement of the PortSwigger Academy material)
- **Target type:** Generic web apps: coupon redemption, fintech withdrawals, resource allocation limits, password reset, shopping carts.
- **Recon signal:** Sequential vs parallel behaviour differences on limited or single-use actions; inconsistent response times across endpoints in multi-endpoint attacks.
- **Root cause:** Multiple threads read and write shared data with a gap between check and use (the author's movie-ticket analogy); a sub-state exists between the start of processing and the database update.
- **Exploitation steps:**
  1. Limit overrun: send many parallel copies of a single-use action (apply code, withdraw, allocate). Use more than two requests in discovery to absorb server-side jitter and raise the hit chance.
  2. Sync technique by protocol: HTTP/1 uses last-byte sync (send all but the final byte of each request, then release the last bytes together); HTTP/2 uses the single-packet attack (complete requests combined into a single TCP packet).
  3. Turbo Intruder single-packet setup: confirm HTTP/2, then `RequestEngine(endpoint=target.endpoint, concurrentConnections=1, engine=Engine.BURP2)`, queue 20 requests with `gate='1'`, release with `engine.openGate('1')`.
  4. Caution on the article's other snippet: it shows `concurrentConnections=20, requestsPerConnection=100, pipeline=False` with a gate but no `openGate` call and no BURP2 engine, so it would not perform a single-packet attack. Treat as illustrative only and prefer the BURP2 / concurrentConnections=1 / gate / openGate pattern.
  5. Methodology: predict (is it security critical, is there collision potential on the same record), probe (benchmark sequentially, then send in parallel, look for any deviation including emails and UI changes), prove (understand, remove superfluous requests, then escalate).
  6. Single-endpoint example: two parallel resets from one session with different usernames, where the session holds both user id and token.
  7. Multi-endpoint alignment: separate connection-related delay (affects the first request only, can be ignored) from endpoint-specific delay (persists after warming). Workarounds: shift slower endpoints earlier, reuse connections or use HTTP/2, pad faster endpoints (delays or larger payloads), send warm-up requests in Turbo Intruder. If warming fails, use a client-side delay (loses single-packet reliability, bad on high-jitter targets) or trigger a server-side delay by flooding dummy requests into a rate limit and then sending the real requests.
- **Bypasses used:** single-packet attack, last-byte sync, connection warming, client-side delay, rate-limit-induced server delay, request padding of faster endpoints.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Secondary summary; the technical content matches the PortSwigger topic page. Value is the plain explanation of why to send 20-30 requests (jitter, hit chance, discovery) and the consolidated list of multi-endpoint alignment tricks. Prevention list: single trusted data source, atomic transactions, DB unique and foreign key constraints, do not use sessions to guard DB logic, update session state in one step, careful stateless JWT design.
- **Source:**
  - Title: One Bug, Many Faces: Understanding Every Type of Race Condition Vulnerability
  - Author: Advik Kant (deoxys, DEV Community)
  - URL: https://dev.to/deoxys/one-bug-many-faces-understanding-every-type-of-race-condition-vulnerability-4po3
  - Date: 2025-08-02
  - Disclosed report: none
  - Raw file: `sources/race-conditions/07-devto-deoxys-race-types.md`

Tags: methodology, taxonomy, limit-overrun, multi-endpoint, single-endpoint, single-packet-attack, last-byte-sync, turbo-intruder, connection-warming, rate-limit-bypass, send-group-parallel, predict-probe-prove, defense, reference
