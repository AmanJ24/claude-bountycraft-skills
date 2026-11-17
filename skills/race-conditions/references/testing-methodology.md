# Race condition testing methodology (predict / probe / prove)

From the Kettle whitepaper (note 01) and the PortSwigger topic (note 03). Research/methodology only; authorized, in-scope, test-data, stop-at-proof (see `../../CLAUDE.md`). Delivery technique in `the-single-packet-attack.md`.

## Step 0 - Confirm the delivery technique
- Check HTTP version against the target directly (not through a proxy that may downgrade): `curl -s --http2 -o /dev/null -w '%{http_version}' https://target`. `2` = single-packet attack available; `1.1` = last-byte sync.

## Step 1 - Predict collisions
- List every object that carries a security control: user, session, order, balance, membership, token.
- For each, list the endpoints that **write it** or **read it and act on the result** (register, profile edit, reset start, reset finish, login, redeem, checkout).
- Prune each candidate with the **three questions**:
  1. **Where is the state stored?** Client-side (an emailed JWT reset token, a signed cookie) cannot collide. Server-side persistent state is the ideal target. Session state is partly protected (and may be locked).
  2. **Editing or appending?** Appending (add another email/item) is usually only limit-overrun. Editing a single record (change primary email, decrement balance) has real collision potential.
  3. **What is the operation keyed on?** Two operations must share a key to collide. A reset keyed on each user's own `userid` row cannot be crossed between two victims; a reset whose state lives in the **session** can (token for one email and userid for another end up mixed in one session).

## Step 2 - Probe
- **Benchmark first.** Send the group **sequentially** (Burp "Send group in sequence (separate connections)", seconds apart) and record the normal responses. You cannot recognize an anomaly without this baseline.
- **Then send the same group in parallel** (single-packet attack, or last-byte sync). During discovery use **20-30 copies** to overcome server-side jitter (CPU contention), even though the final exploit usually needs only two.
- **Hunt any deviation** from the baseline: different status or body, an extra or misrouted email, a changed session, a 500 error, a suspiciously fast response (work handed to another thread) or slow one (locking/resource limit). Deviations in **second-order effects** (email contents, later state) count; deferred races show no immediate response difference at all (note 01).
- If every parallel request still processes **sequentially**, suspect **session locking** (PHP native sessions). Re-run with a **distinct session token per request**.

## Step 3 - Prove
- Reduce to the **minimum requests** (usually two) that still reproduce. Extra requests can overwrite a good end state, so fewer is both cleaner and sometimes necessary.
- Two requests are more timing-sensitive, so **retry / automate** (a Turbo Intruder script; for email flows, `email-link-extraction.py` triggers the emails, pulls them from Collaborator, extracts and visits the links).
- **Understand the mechanism**, then escalate: treat the race as a structural weakness and find the maximum-impact primitive (two endpoints disagreeing on your email -> verified-email forgery -> account takeover).

## Multi-endpoint window alignment
When two **different** endpoints must collide (payment vs cart, confirm vs change):
1. Benchmark to separate **connection delay** (first request on a fresh back-end connection is slow because of setup; ignore it) from **endpoint-processing delay** (persists). Use "Send group in sequence (single connection)" with a leading homepage GET.
2. If one endpoint stays inconsistent even under single-packet, add **connection-warming** requests (inconsequential GETs down the connection first) in Turbo Intruder, because front-ends route per connection and may open fresh back-end connections.
3. If warming does not help: prefer a **server-side delay** by flooding dummy requests to trip a rate/resource limit on the slower endpoint (this keeps the single-packet attack viable) over a **client-side delay** (which splits requests across packets and loses single-packet reliability on high-jitter targets). Add two copies of the "attack" request so a second gets another chance to land in the window.

## Sub-variant-specific tactics
- **Limit overrun**: N parallel copies of one single-use/limited action.
- **Single-endpoint collision**: two parallel requests from the **same session** with different values (two email changes, two resets) where the session stores the two fields separately.
- **Partial construction**: during registration, race an authenticated request against the uninitialized security field with a value equal to its empty state (PHP `api-key[]=`; Rails `param[key]` -> nil).
- **Time-sensitive**: fire precisely-timed requests so two operations produce the same timestamp-derived token.
- **MFA sub-state**: login in parallel with a request to a sensitive authenticated page.

## Pitfalls
- No sequential baseline = you cannot tell a race from noise.
- Client-side state and cross-key operations cannot collide (false-positive guards; see SKILL.md section 5).
- A static file / non-dynamic endpoint breaks single-packet (negative timestamp); target dynamic endpoints.
- Race patches are frequently incomplete; always retest with concurrency after a fix.
