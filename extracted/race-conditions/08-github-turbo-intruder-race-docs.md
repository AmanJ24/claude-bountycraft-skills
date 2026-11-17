# Turbo Intruder gated-request mechanics for race testing

- **Vuln type / sub-variant:** Methodology/technique reference (tooling for single-packet, multi-connection and HTTP/3 race delivery, plus a timing-comparison pattern)
- **Target type:** Any HTTP endpoint where concurrent requests may collide (HTTP/2, HTTP/1.1, HTTP/3 servers)
- **Recon signal:** Not applicable. This is the tooling doc a tester reaches for once a race-prone endpoint has been identified.
- **Root cause:** Not a finding. The doc exists because network jitter makes naive parallel sending unreliable, so requests must be held back and released together.
- **Exploitation steps:**
  1. Build a RequestEngine for the endpoint and queue N copies of the request (the doc's example uses 20), each tagged with the same `gate` name so they are held, not sent.
  2. Call `openGate('<name>')` to release the whole group at once.
  3. For HTTP/2 targets use `engine=Engine.BURP2` with `concurrentConnections=1`. All gated requests then go out in a single TCP packet, which is the single-packet attack.
  4. For HTTP/1.1 targets use `Engine.THREADED` (or `Engine.BURP`) with `concurrentConnections` equal to the request count, so many connections fire as close together as possible.
  5. For HTTP/3 use `Engine.HTTP3` with `gateMode='auto'`. It picks the QPACK gate when the server advertises both needed settings, else a single-datagram gate. `'sda'` or `'qpack'` forces one.
  6. For multi-endpoint races, queue different requests under separate gates and open them in sequence (the example opens gate1, sleeps 0.1s, opens gate2). A bundled multi-endpoint example script exists.
  7. Read results: a negative timestamp in the results table means the server answered before the request was fully sent, a sign of overlapping server-side processing. `req.order == 0` marks the response that came back first (the "winner"); order callbacks fire only after the whole gate batch returns.
  8. For timing-difference testing (not a classic race), alternate payload order per gate across about 100 repeats, with two connections and a short sleep (0.2s) between gates to avoid rate limiting, then compare the labeled response times statistically.
- **Bypasses used:** Engine choice per protocol. Alternating left/right queue order each repeat to avoid order-bias false positives in timing comparisons. Sleeping between gates to dodge rate limiting.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Gate requests, then release them together. Pick the engine by protocol: BURP2 with one connection for HTTP/2 single-packet, threaded multi-connection for HTTP/1.1, HTTP3 gate modes for QUIC. Use response order and negative timestamps as race signals.
- **Source:**
  - Title: Race Condition Testing (Turbo Intruder docs)
  - Author: PortSwigger (turbo-intruder repo)
  - URL: https://raw.githubusercontent.com/PortSwigger/turbo-intruder/master/docs/race-conditions.md
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/race-conditions/08-github-turbo-intruder-race-docs.md`

Tags: methodology, tooling, turbo-intruder, single-packet-attack, send-group-parallel, benchmarking, reference
