# First Sequence Sync: scaling the single-packet attack past 20-30 requests to ~10,000

- **Vuln type / sub-variant:** methodology/technique (expert extension of the single-packet attack for large-scale limit-overrun and rate-limit/OTP-brute-via-race)
- **Target type:** HTTP/2 servers where a limit-overrun or a one-time-token/PIN rate limit needs far more than 20-30 synchronized requests to break reliably
- **Recon signal:** A limit-overrun or token-brute scenario that the standard single-packet attack cannot win because it needs thousands of simultaneous requests (e.g. brute-forcing a numeric one-time token whose rate limit caps attempts at 5), and the single-packet attack is capped at ~20-30 requests / ~1,500 bytes.
- **Root cause:** The single-packet attack packs all requests into one TCP packet, which is bounded by the ~1,500-byte Ethernet MTU (and the 65,535-byte TCP limit). That caps synchronized requests at ~20-30. RyotaK's technique removes both caps so one synchronized burst can carry ~10,000 requests.
- **Exploitation steps (the technique):**
  1. Open one HTTP/2 connection and open a stream per request; send each request except its final byte, so the server holds them unprocessed (the last-byte-sync idea, scaled up).
  2. **IP fragmentation**: build large TCP packets holding many HTTP/2 frames (the withheld final bytes) and split each over multiple IP fragments. The server will not hand a TCP packet to the TCP layer until all its IP fragments arrive, so you can assemble a packet far larger than one Ethernet frame, up to the full TCP window (~62-65 KB observed).
  3. **First Sequence Sync**: give the TCP packets out-of-order sequence numbers and send every packet EXCEPT the one holding the first sequence number. Because TCP processes segments in sequence-number order, the server queues them all and processes none until the first-sequence packet arrives.
  4. Send the first-sequence packet last. The server now has every fragment and reassembles/processes all requests at once, synchronized.
- **Bypasses used:** IP-layer fragmentation (break the 1,500-byte MTU), TCP sequence-number reordering (hold processing until the final packet), last-byte sync at the HTTP/2 layer. On the client, drop outbound RST (`iptables ... --tcp-flags RST RST -j DROP`) so the kernel does not tear down the hand-built packets.
- **Impact:** n/a (reference). Demonstrated: 10,000 synchronized requests in ~166ms across ~250ms-latency hosts (~0.0166ms/request); turned a 5-attempt OTP rate limit into ~1,000 attempts in one race (vs ~10 with last-byte sync), i.e. a reliable one-time-token authentication bypass.
- **Limiting factors (what to check first):** the server's `SETTINGS_MAX_CONCURRENT_STREAMS` caps requests per connection (Apache 100, Nginx 128, Go 250), but the RFC default is unlimited and some stacks set it huge (nghttp2 / Node.js ~4.29e9), so the technique is only as powerful as the target's stream limit. Also needs enough server TCP buffer (usually fine). Current PoC is HTTP/2-cleartext and breaks if the server updates its TCP window mid-send.
- **Bounty:** n/a (reference).
- **Lesson / pattern:** When a limit-overrun or token-brute needs thousands of synchronized requests, the plain single-packet attack is not enough; first-sequence-sync (IP fragmentation + sequence reordering) is the expert escalation. Before attempting it, check the target's `SETTINGS_MAX_CONCURRENT_STREAMS`, because that, not the attacker, sets the real concurrency ceiling. This operates at OSI layers 3-4, so Burp/Caido (layer 7) cannot do it today; it needs the custom tooling.
- **Source:**
  - Title: Beyond the Limit: Expanding single-packet race condition with a first sequence sync
  - Author: RyotaK (Flatt Security)
  - URL: https://flatt.tech/research/posts/beyond-the-limit-expanding-single-packet-race-condition-with-first-sequence-sync/
  - Date: 2024-08-02
  - Disclosed report: none
  - Raw file: `sources/race-conditions/04-flatt-ryotak-first-sequence-sync.md`

Tags: methodology, limit-overrun, rate-limit-bypass, otp-brute, first-sequence-sync, single-packet-attack, last-byte-sync, benchmarking, reference
