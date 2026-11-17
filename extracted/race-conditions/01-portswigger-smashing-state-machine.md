# Smashing the state machine: sub-states, single-packet attack, predict/probe/prove, and new race classes

- **Vuln type / sub-variant:** methodology/technique (also the foundational taxonomy: limit-overrun, object masking, multi-endpoint, single-endpoint, deferred, partial construction)
- **Target type:** Any multi-threaded web app with shared state (DB, session store, ORM). Case studies: GitLab (email invitations, email confirmation via Devise/Rails), an unnamed major site with batched email jobs, a custom session handler hypothetical.
- **Recon signal:**
  - Core idea: every HTTP request is multi-step, because it walks the app through short-lived hidden "sub-states" (typically ~1ms race windows).
  - Clues are any deviation from benchmarked sequential behavior: different response bodies or codes, extra or misrouted emails, a changed session, 500 errors, a surprisingly fast response (suggests work handed to another thread), or a slow one (suggests locking or resource limits).
  - Deferred races produce no immediate clue, only later anomalies (mismatched emails, odd state).
- **Root cause:**
  - Non-atomic request handling: a check and the action it protects are separated in time (TOCTOU), or state is built/edited in several writes.
  - Mixed data sources: Devise sent the confirmation email to an address passed in an in-memory argument, but rendered the body (including the token) by re-reading the DB in another thread. A second request changing `unconfirmed_email` in between made recipient and token disagree.
  - Unsafe session/ORM strategies: locking (PHP native sessions serialize same-session requests), batching (load whole record, write back at the end, last writer wins), or no defence (custom redis/db session handlers that write variables individually).
- **Exploitation steps:**
  1. PREDICT: list objects with security controls (users, sessions, orders). For each, list endpoints that write it or read it and act on the data (registration, profile edit, reset start, reset finish, login).
  2. Prune with three questions. (a) Where is state stored? Client-side state (e-mailed JWT reset) cannot collide; server-side persistent state is ideal; session state is partly protected. (b) Editing or appending? Appending (add another email) is mostly only limit-overrun; editing (change primary email) has real collision potential. (c) What is the operation keyed on? Two operations need the same key. Reset keyed on userid in the users table cannot collide across two victims; reset keyed on sessionid can (token and userid for different emails end up mixed in one session).
  3. PROBE: build a blend of requests hitting all relevant code paths several times with different values. Benchmark first by sending them sequentially with seconds between each. Then fire the whole blend at once with the single-packet attack (last-byte sync if no HTTP/2). Compare to baseline and hunt for any deviation. Use many requests here to beat server-side jitter. Use a separate session per request when the framework locks on session id.
  4. PROVE: reduce to the minimum requests (usually two) that still reproduce, because extra requests can overwrite a good end state. Two requests are more timing sensitive, so retry or automate (Turbo Intruder script that triggers emails, pulls them from Collaborator, extracts and visits links; `email-link-extraction.py`). Then escalate: treat the race as a structural weakness and look for what primitive it gives (for example endpoints disagreeing on your email).
  5. Case: GitLab invitations. Send 6 identical invite requests. Sequential gives 6 success and 1 email; parallel gave 1 success, 5 "email already taken" and 2 emails. That is object masking: the list page shows one invitation per email, so a low-privilege decoy can be replaced by an admin-level one on revoke.
  6. Case: GitLab single-endpoint (CVE-2022-4037). Two parallel `POST /-/profile` changing `user[email]` to two different addresses. The email sent to address B could carry a body and token for address A. For a valid token the app must be in the "resend existing token" path (hit resend confirmation endpoint, or request a change to the same address twice). Using your own address and a target-company address yields a verified email you do not own, which enables invitation hijack and third-party account takeover through GitLab acting as OpenID IdP.
  7. Case: multi-endpoint email verify vs change. Confirmation request always ran before email-change, so a 90ms client delay on the confirmation gave a ~50/50 ordering. Client-side delay breaks single-packet, so alternatively send a burst of dummy requests to trigger a leaky-bucket rate limit that delays processing server-side.
  8. Case: deferred collision. Two email-change requests sent up to ~20 minutes apart still collided, because the site batch-processed emails later. Spot via anomalies (two emails to one address), not response differences.
  9. Hypothetical vulnerable session-handler snippets to test: reset flow storing username and code in session (race two resets); login setting `session['user']` before `require2fa` (race login with a sensitive page fetch); session-swap (force a victim onto your anonymous session cookie, then race logins).
- **Bypasses used:**
  - Single-packet attack: send all HTTP/2 requests over one connection with the final frame/byte of each withheld, wait ~100ms, then flush the final frames together so they leave in one TCP packet.
  - Withheld part: for bodyless requests, send headers without END_STREAM and withhold an empty DATA frame carrying END_STREAM; for requests with a body, send headers plus all body except the last byte and withhold a DATA frame with that byte. Sending the full body and only withholding END_STREAM breaks servers that use content-length to decide completeness.
  - Implementation details: leave Nagle's algorithm on (TCP_NODELAY disabled) so final frames coalesce; send a PING first to warm the local connection so the first final frame is not split into its own packet; verify in Wireshark. No custom TCP/TLS stack needed. Works on dynamic endpoints; static files on some servers break (negative timestamp in Turbo Intruder, which doubles as a static-file test).
  - Benchmark (Melbourne to Dublin, 20 requests): last-byte sync median spread 4ms (SD 3ms), single-packet 1ms (SD 0.3ms). On one real replication single-packet succeeded in ~30s while last-byte took over 2 hours.
  - Single-packet carries roughly 20-30 requests (about the 1,500 byte TCP soft limit).
  - Connection warming: send inconsequential requests down the connection first, because front-ends route per connection and may open fresh back-end connections for some requests.
  - Client-side delay (breaks single-packet) or rate-limit abuse (dummy flood causes server-side delay) for aligning multi-endpoint windows.
  - Distinct sessions per request to defeat PHP-style session locking.
  - Last-byte sync fallback for HTTP/1.1.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** With race conditions everything is multi-step. Predict collisions by state location, edit-vs-append and key, benchmark sequentially before blasting in parallel, treat any anomaly as a clue, then shrink to two requests and escalate. Anomaly spotting is the key skill. Also: the author notes race patches are often incomplete, and that "defence" means atomic transactions, uniqueness constraints and not mixing data stores.
- **Source:**
  - Title: Smashing the state machine: the true potential of web race conditions
  - Author: James Kettle (PortSwigger Research)
  - URL: https://portswigger.net/research/smashing-the-state-machine
  - Date: 2023-08-09
  - Disclosed report: none (CVE-2022-4037 for the GitLab bug)
  - Raw file: `sources/race-conditions/01-portswigger-smashing-state-machine.md`

Tags: methodology, taxonomy, single-packet-attack, last-byte-sync, predict-probe-prove, sub-state, multi-endpoint, single-endpoint, partial-construction, connection-warming, rate-limit-bypass, benchmarking, send-group-parallel, turbo-intruder, auth-flow, state-corruption, account-takeover, toctou, defense
