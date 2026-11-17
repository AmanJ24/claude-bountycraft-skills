# Single-packet attack explained: HTTP/2 frames, Nagle, last-byte sync (H2SpaceX)

- **Vuln type / sub-variant:** Methodology/technique reference (how the HTTP/2 single-packet attack removes network jitter)
- **Target type:** HTTP/2 servers where a race window exists
- **Recon signal:** Not applicable. Written as a technique explainer after the author watched James Kettle's DEF CON 31 talk, plus a custom tool (H2SpaceX) as an alternative to Turbo Intruder's SpikeEngine.
- **Root cause:** Not a finding. The core problem addressed is jitter: variance in packet delay means separately sent packets do not reach the server together, so race windows are hit unreliably.
- **Exploitation steps:**
  1. Understand the building blocks. HTTP/2 multiplexes requests as frames tagged with stream IDs, so many requests can share one TCP segment without head-of-line blocking.
  2. Recognize the size limit: roughly 1500 bytes per packet, so large POST bodies mean only one request fits.
  3. Use last-byte sync to shrink what must fit: send every request except its final byte, then release the final bytes together.
  4. Enable Nagle's algorithm (disable TCP_NODELAY) so small writes coalesce into one packet. Together with last-byte sync this fits about 20 requests in one packet.
  5. Frame-level recipe: build HEADERS plus DATA frames per stream; strip END_STREAM from the last DATA frame and hold back a new DATA frame carrying the last byte with END_STREAM set; wait about 100 ms; send a PING frame to warm the connection; then send all held final-byte frames together.
  6. Automate with H2SpaceX, which allows reading responses and deciding follow-up actions, and ships examples for PortSwigger race labs.
- **Bypasses used:** Last-byte sync plus Nagle coalescing to get past the per-packet size limit. PING frame for connection warming so the final burst is not delayed by connection setup.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** The single packet is what eliminates jitter. Fit many requests into one TCP segment by holding back each request's last byte, warming the connection with a PING, then releasing everything at once.
- **Source:**
  - Title: Dive into Single Packet Attack
  - Author: Amin Nasiri (nxenon)
  - URL: https://infosecwriteups.com/dive-into-single-packet-attack-3d3849ffe1d2
  - Date: 2023-11-24
  - Disclosed report: none
  - Raw file: `sources/race-conditions/09-medium-amin-nasiri-single-packet.md`

Tags: methodology, tooling, single-packet-attack, last-byte-sync, connection-warming, reference
