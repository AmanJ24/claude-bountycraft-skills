# The delivery technique: single-packet attack, last-byte sync, first-sequence-sync

Race windows are often ~1ms. Network jitter (uneven packet arrival) historically made remote races unreliable. The techniques here make the server see your requests at the same instant. Authorized targets and test data only.

## Why jitter hides races
Requests sent "at the same time" arrive unevenly because of network delay variance, so they miss a ~1ms window. The fix: make all requests *complete* on the server simultaneously, regardless of when their bulk arrived (notes 01, 02, 09).

## Single-packet attack (HTTP/2, the default)
Mechanics (notes 01, 09):
1. Open one HTTP/2 connection and start 20-30 requests, but **withhold a tiny final fragment** of each so the server will not begin processing. For a bodyless request, send HEADERS without END_STREAM and hold back an empty DATA frame carrying END_STREAM. For a request with a body, send headers plus all body except the last byte, and hold back a DATA frame with that last byte. (Withholding only END_STREAM breaks servers that use Content-Length to judge completeness.)
2. Wait ~100ms for the bulk to arrive.
3. **Release all final fragments together.** The OS packs them into one TCP packet, so the server completes and processes them simultaneously.
Implementation details: leave Nagle's algorithm on (do not set TCP_NODELAY) so the small final frames coalesce; send a PING first to warm the connection so the first final frame is not split off; verify in Wireshark if hand-rolling. Benchmarks: ~1ms spread (SD 0.3ms) vs 4ms for last-byte sync; one real bug fell in ~30s with single-packet vs 2+ hours with last-byte sync. Capacity: ~20-30 requests (the ~1,500-byte Ethernet/TCP soft limit).

## Last-byte sync (HTTP/1.1 fallback)
HTTP/1.1 forces responses in request order (head-of-line blocking), so requests on one connection process serially. Instead, open **parallel connections**, send all but the last byte of each request, then release the last bytes together (note 02). Burp Repeater and Turbo Intruder pick this automatically for HTTP/1.1.

## Protocol applicability (note 02)
- **HTTP/2**: single-packet attack. Primary target.
- **HTTP/1.1**: parallel connections + last-byte sync.
- **HTTP/3 (QUIC)**: possible (multiplexed streams in a UDP datagram) but datagram ~1,500 bytes caps it similarly; low priority.
- **WebSocket**: promising (no response ordering) but one connection cannot interleave fragments of different messages (RFC 6455), so you cannot withhold many at once; RFC 8441 (WebSocket over HTTP/2) would restore it. Under-tested for lack of tooling.
- **SMTP / pipelined HTTP/1.1**: in-order responses, effectively serial.
- General rule to adapt: you need multiplexing, interleaved fragments, and a large max packet size; always build a PoC and test the real server.

## Tooling config
- **Burp Repeater**: put the requests in a tab group, choose **"Send group in parallel"** (Burp 2023.9+ picks single-packet for HTTP/2, last-byte sync for HTTP/1). "Send group in sequence (separate connections)" is the sequential baseline; "Send group in sequence (single connection)" + a leading GET is the connection-warming benchmark. A Bambda "Trigger race conditions" custom action fires a parallel probe in one click.
- **Turbo Intruder** (note 03, 08): HTTP/2 single-packet = `RequestEngine(endpoint=target.endpoint, concurrentConnections=1, engine=Engine.BURP2)`, then `engine.queue(target.req, gate='1')` in a loop (e.g. 20x) and `engine.openGate('1')` to release. Template: `race-single-packet-attack.py`. For HTTP/1.1-only use `Engine.THREADED`/`Engine.BURP` with `concurrentConnections` = request count. For HTTP/3 use `Engine.HTTP3` with `gateMode='auto'`. **Force last-byte sync on an HTTP/2 server** (rare) with `Engine.THREADED`, since BURP2 would multiplex everything onto one connection.
- **Multi-endpoint** in Turbo Intruder: queue different requests under separate gates and open them with a tiny stagger, or open together; set `requestsPerConnection` to the number of gated requests; set endpoint/host manually per request.
- **Fit more requests per packet**: strip non-essential headers (Accept, Accept-Language, Accept-Encoding, User-Agent). Each HTTP/2 frame costs a 9-byte header + compressed headers; lean requests fit 20-30.
- **Signals**: a **negative timestamp** in Turbo Intruder = the server answered before the request was complete (also a static-file test; static files on some servers break single-packet). `req.order == 0` marks the first response back.

## First sequence sync (expert escalation, note 04)
When a limit-overrun or OTP-brute needs **thousands** of synchronized requests (beyond the ~20-30 cap), RyotaK's technique:
1. Last-byte-sync the HTTP/2 layer (withhold each request's final byte).
2. **IP fragmentation**: split each large TCP packet across multiple IP fragments; the server will not pass a packet to TCP until all fragments arrive, breaking the ~1,500-byte MTU up to the full TCP window (~62-65KB).
3. **First sequence sync**: give the TCP packets out-of-order sequence numbers and send every one except the packet holding the first sequence number; TCP queues them unprocessed. Send the first-sequence packet last to release everything at once.
Drop outbound RST on the client (`iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP`). Demonstrated ~10,000 requests in ~166ms; turned a 5-attempt PIN limit into ~1,000 attempts. The real ceiling is the server's `SETTINGS_MAX_CONCURRENT_STREAMS` (Apache 100, Nginx 128, Go 250; nghttp2/Node.js effectively unlimited). Operates at OSI layers 3-4, so Burp/Caido cannot do it; needs custom tooling.

## Harness limitation (this project)
In-page `fetch` with `Promise.all` sends requests but cannot do last-byte sync or single-packet coalescing, so it only wins **wide** race windows and is unreliable for the sub-millisecond ones the labs use. Treat an in-harness non-win as a tooling limit, not evidence the target is safe.
