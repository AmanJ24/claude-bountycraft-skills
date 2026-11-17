# Single-packet attack beyond HTTP/2: protocol-by-protocol viability analysis

- **Vuln type / sub-variant:** methodology/technique (protocol-level analysis of where single-packet synchronization works)
- **Target type:** Any server speaking a multiplexed or pipelined protocol: HTTP/2 (primary), HTTP/3 over QUIC, HTTP/1.1 pipelining, WebSocket, SMTP.
- **Recon signal:** Not a finding. The practical decision signal is which protocol the target negotiates, since that decides which synchronization primitive to use. Multiplexing support on one connection is the enabling feature.
- **Root cause:** Network jitter makes simultaneously sent requests arrive at uneven times and hides small race windows. The fix is to make the server see all final request fragments at once, which needs multiple in-flight messages on one connection plus a way to withhold completion.
- **Exploitation steps:**
  1. Recap of mechanics: open one HTTP/2 connection and start all requests but hold back a tiny fragment of each so the server will not begin processing. Wait briefly for the bulk to land. Release the final fragments together so the OS packs them into one TCP packet. Scales to about 20-30 requests. Available in Burp Repeater (tab group, send in parallel) and Turbo Intruder.
  2. HTTP/3: possible in principle (multiplexed streams in one UDP datagram) but a datagram caps near 1,500 bytes. TCP has the same practical soft limit, so no real loss, but not worth building for now. A better investment is pushing HTTP/2 closer to the TCP maximum packet size, which could mean around 800 simultaneous requests.
  3. HTTP/1.1: pipelining can put several full requests in one packet, but the RFC forces responses in request order (head-of-line blocking), so servers mostly process sequentially. Use parallel connections with last-byte sync instead; Burp Repeater does this automatically for HTTP/1.1.
  4. SMTP: pipelining (RFC 2920) has the same in-order response problem, so unlikely to process in parallel, even though SMTP handlers probably hide races.
  5. WebSocket: promising because there is no response concept, so servers can process messages concurrently. But fragments of different messages cannot be interleaved on one connection (RFC 6455), so you cannot withhold many messages at once. RFC 8441 (WebSocket bootstrapped over HTTP/2 streams) would restore the full attack where servers support it. WebSocket races are under-tested mostly for lack of tooling.
  6. General rule for adapting to another protocol: you need multiplexing, support for interleaved fragments (raises the message count per packet), and a large maximum packet size. Always build a proof of concept and test against real server implementations.
  7. Alternative coalescing layer: put final fragments into a single TLS record instead of a TCP packet. That should survive non-decrypting tunnels such as SOCKS proxies, but needs a customized TLS stack.
- **Bypasses used:** single-packet attack (HTTP/2); last-byte sync (HTTP/1.1 fallback); proposed TLS-record coalescing and HTTP/2-bootstrapped WebSocket as future variants.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Match the synchronization technique to the protocol. HTTP/2 gets single-packet; HTTP/1.1 gets parallel connections with last-byte sync; WebSocket is an under-explored race surface; SMTP and pipelined HTTP/1.1 are effectively serial. Packet size (not request count logic) is the ceiling on simultaneous requests, which motivated later work like first-sequence sync.
- **Source:**
  - Title: The single-packet attack: making remote race-conditions 'local'
  - Author: James Kettle (PortSwigger Research)
  - URL: https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local
  - Date: 2023-10-18
  - Disclosed report: none
  - Raw file: `sources/race-conditions/02-portswigger-single-packet-attack-local.md`

Tags: methodology, technique, single-packet-attack, last-byte-sync, send-group-parallel, turbo-intruder, reference, tooling
