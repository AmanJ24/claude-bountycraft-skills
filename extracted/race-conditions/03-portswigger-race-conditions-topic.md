# Web Security Academy race conditions topic: taxonomy, tooling and methodology reference

- **Vuln type / sub-variant:** methodology/taxonomy (limit-overrun, hidden multi-step/sub-state, multi-endpoint, single-endpoint, session-locking masking, partial construction, time-sensitive)
- **Target type:** Generic web apps: discount/gift-card/redeem endpoints, MFA login, checkout flows, password reset, email confirmation and other email-driven flows, user registration with API keys, token generation.
- **Recon signal:**
  - A single-use or rate-limited endpoint with security impact (limit-overrun).
  - Security-critical endpoints where two requests can touch the same record.
  - Emails sent from background threads after the HTTP response is returned (single-endpoint races).
  - All requests processing sequentially despite parallel send, which signals session-based locking (for example PHP's native session handler).
  - Security tokens derived from high-resolution timestamps.
- **Root cause:**
  - Concurrent processing with no safeguards: the "race window" lies between a check and a state update (a sub-state the app enters and leaves within one request).
  - Multi-step operations hidden in one request, for example `session['userid']` set, then `enforce_mfa` set afterwards, leaving a logged-in-but-MFA-not-enforced sub-state.
  - Objects built in several statements (user row inserted, then API key set), leaving uninitialized fields.
  - Weak randomness: a timestamp-only reset token collides when two resets land on the same timestamp.
- **Exploitation steps:**
  1. Limit overrun in Repeater: identify limited endpoint, group many copies of the request in a tab group, use "Send group in parallel". Burp picks last-byte sync for HTTP/1 and single-packet for HTTP/2 (Burp 2023.9+). Sending many (20-30) copies helps with server-side jitter during discovery, though two usually suffice for the exploit.
  2. Limit overrun in Turbo Intruder (needs HTTP/2): set `engine=Engine.BURP2` and `concurrentConnections=1` on `RequestEngine(endpoint=target.endpoint, ...)`. Queue the requests with a named gate argument, e.g. loop 20 times `engine.queue(target.req, gate='1')`, then release them with `engine.openGate('1')`. Template: `race-single-packet-attack.py`. Use Turbo Intruder over Repeater when you need retries, staggered timing or very many requests.
  3. Methodology predict/probe/prove (summarized from the whitepaper): (a) Predict: is the endpoint security critical, and is there collision potential (two requests operating on the same record)? Example: reset keyed on user record cannot collide across two users; reset state stored in the session can. (b) Probe: benchmark with "Send group in sequence (separate connections)", then send the same group with "Send group in parallel", and look for any deviation including second-order effects such as different email content. A Bambda custom action "Trigger race conditions" can fire a parallel probe in one click. (c) Prove: understand the mechanism, strip superfluous requests, confirm it still reproduces, and look for the maximum-impact primitive.
  4. MFA sub-state: fire the login request in parallel with a request to a sensitive authenticated endpoint, to land in the window where the session is valid but MFA is not yet enforced.
  5. Multi-endpoint (for example payment vs add-to-cart): race the add-item request into the window between payment validation and order confirmation. To align windows, first work out whether delay is connection setup or endpoint processing: put a homepage GET at the front of a tab group and use "Send group in sequence (single connection)". If only the first request is slow, it is back-end connection delay and can be ignored. If one endpoint stays inconsistent even under single-packet, add connection-warming requests in Turbo Intruder before the attack.
  6. If warming does not help: add a client-side delay in Turbo Intruder (this splits requests across packets, so it loses single-packet reliability on high-jitter targets) or flood dummy requests to trip a rate or resource limit and cause a server-side delay, which keeps single-packet viable.
  7. Single-endpoint: send two parallel reset requests from the same session with different usernames; if session stores reset-user and reset-token separately, the end state can be victim's user id with the token mailed to the attacker. Needs correct operation ordering, so expect multiple attempts. Email confirmation and other email-based operations are good targets.
  8. Session locking: if everything serializes, use a different session token per request.
  9. Partial construction: during registration race an authenticated request against the uninitialized field with a value that equals the uninitialized state. PHP: `param[]=foo` is an array and bare `param[]` is an empty array. Rails: `param[key]` yields `{"param"=>{"key"=>nil}}`. Example: `GET /api/user/info?user=victim&api-key[]=`. For hashed passwords you need an input whose hash digest equals the uninitialized value.
  10. Time-sensitive: use precisely timed requests to make two resets for different users produce the same timestamp-derived token.
- **Bypasses used:** single-packet attack (HTTP/2), last-byte sync (HTTP/1), connection warming, client-side delay, rate/resource-limit-induced server delay, per-request session tokens against session locking, array/null input to match uninitialized values.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Limit-overrun is only the simplest class. For anything else, find two operations keyed on the same record, benchmark sequentially first, then probe in parallel and chase any deviation. Check for session locking whenever requests look serialized. Defences listed: atomic datastore operations (one transaction), uniqueness constraints as defence in depth, never use one storage layer to protect another, keep session/ORM updates batched and consistent, optionally move state client-side.
- **Source:**
  - Title: Race conditions | Web Security Academy
  - Author: PortSwigger (Web Security Academy)
  - URL: https://portswigger.net/web-security/race-conditions
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/race-conditions/03-portswigger-race-conditions-topic.md`

Tags: methodology, taxonomy, limit-overrun, multi-endpoint, single-endpoint, partial-construction, time-sensitive, single-packet-attack, last-byte-sync, turbo-intruder, send-group-parallel, connection-warming, rate-limit-bypass, predict-probe-prove, sub-state, benchmarking, auth-flow, toctou, defense, reference
