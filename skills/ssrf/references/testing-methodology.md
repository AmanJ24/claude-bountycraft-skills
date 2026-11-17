# SSRF testing methodology (step by step)

Research/methodology guidance. Against a real target, only proceed under authorized, in-scope conditions; probe your own hosts/ports before any real internal or metadata address (see `../../CLAUDE.md`).

## Step 1 - Map every server-side fetch
- Walk the app and list parameters and features that could make the backend request something. Use the parameter quick-list in `attack-surfaces.md`.
- Inspect **background requests**: intercept save/submit/API calls behind "preview", "print", "generate", "import", "test". The DoD and HackerOne PDF bugs lived in JSON save calls, not visible form fields (notes 05, 01).
- Check **error and not-found paths**, not just valid input. The $25k bug was a value reflected into a "template not found" message that was later rendered (note 01).
- For transform features (PDF, screenshot, image, video), remember there may be no URL field: the injection point is markup inside your input.

## Step 2 - Confirm server-side origin with one OOB callback
- Use your own Collaborator / interactsh / webhook.site / requestbin host.
- Submit it in the candidate field (or as an `<iframe>`/`<img>`/`xlink:href`/playlist entry for renderers).
- A DNS lookup or HTTP request arriving **from the target's infrastructure** (not your browser IP) confirms SSRF.
- **Read the callback User-Agent.** It fingerprints the engine and chooses your next moves: `Lavf/...` = FFmpeg (note 07); `wkhtmltopdf`, `PhantomJS`, `HeadlessChrome`, `Prince`, `WeasyPrint` = renderer-specific features (note 27); a cloud LB/proxy string hints at the environment.

## Step 2b - OOB confirmation discipline (confirm before you claim)
A callback is the only thing that proves server-side fetch. Treat "the server processed my URL" as a hypothesis until an out-of-band hit lands. This is the single biggest separator between credible SSRF and an N/A at triage.

- **Echo is not confirmation.** A URL reflected back in an error ("the application at `http://you/x` could not be found"), a different status code for an external vs loopback host, or a slower response are all producible by a URL *validator* that never makes a request. Our false-positive list says the same thing; the discipline here is to never report until the callback fires. Blind cases in this set (Sentry note 06, PortSwigger PDF note 26) were only believed after the OOB hit.
- **Attribute the callback to exactly one parameter.** A hit proves *a* request left the server, not *which* field caused it, and the fix depends entirely on that. If you batch several candidate fields into one test and a callback arrives, you cannot say which one fired, and you will have to retest. Use a **fresh, unique OOB host per candidate parameter** and send one request per host. Note that some Collaborator clients key interactions by payload ID, not by subdomain, so per-sink sub-tags from one payload can be indistinguishable: generate a separate payload per candidate.
- **Keep the negative controls.** A parameter that produces no callback is evidence too. It is what lets the program fix the real sink instead of allowlisting an inert field. Record the fields you tested that did *not* fire.
- **Blind vs full-read, decided with one request.** Before scoring, fetch a known, recognizable external body (e.g. a page whose title you know) and check whether that body comes back to you. Body returned = full-read (read internal endpoints directly, much higher severity). Only the callback, no body = blind (low on its own until you reach an internal service or get data back). This maps to the visibility classes in Step 3.
- **A distinct status on a link-local address is itself a signal.** If `169.254.169.254` returns a different status/error than every other host you try, that difference is the metadata service answering, which proves reach to a non-routable address that a callback alone cannot.

## Step 3 - Classify visibility
- **Full read:** you can see the fetched response body (delivery log, returned import content, generated PDF/PNG that embeds it, echoed "test connection" output). Highest impact, aim here.
- **Partial:** only a title, headers, status, size, or error leaks (link preview title note 02; webhook response headers note 09). Still enough for internal service enumeration.
- **Blind:** only the callback fires, no response returned (Sentry note 06; PortSwigger PDF POST note 26). Use timing and success/failure as a side channel (WooCommerce note 23), or exfiltrate via script/DNS from inside a renderer.

## Step 4 - Prove arbitrary destination and port (own infra first)
- Point at your own server on various ports to show the server will reach arbitrary host:port.
- Note which schemes work (`http`, `https`, `ftp`, `file`, `gopher`, SMB) and which ports are reachable outbound; Shopify's SVG reached all TCP ports but one (note 14).
- For a file-existence/port oracle, use chained references where the second fetch only happens if the first succeeds (SVG double-reference, note 14).

## Step 5 - If internal addresses are filtered, run the bypass ladder
- Confirm a filter exists (obvious internal payload is rejected with an error).
- Work `bypasses.md` in order: redirects (every status code) -> IPv6 -> IP encodings -> DNS rebinding/pointing -> resolver quirks -> Unicode -> alternate protocols -> renderer markup -> second-request gaps.
- After any patch during the engagement, rerun the full ladder (note 23).

## Step 6 - Escalate impact (authorized only)
See `exploitation-and-escalation.md`. Priority order by payout in this set:
1. Cloud metadata credentials (AWS/GCP/Azure), including legacy no-header paths.
2. Chain through a reachable internal service into infrastructure (kube-env, kubelet, debug ports).
3. Local file read (`file://`, FFmpeg, UNC/SMB).
4. Internal port scan / service enumeration.

## Step 7 - Stop at proof and record
- Capture the minimum evidence: one credential (value redacted), one internal service banner/name, the first line of one file, one screenshot of the metadata response. Do not enumerate the whole environment or move laterally.
- Record the exact request, the bypass string, the engine fingerprint, and the response class.
- If the next step would reach deeper internal systems, stop and (for a real engagement) ask the program first. Several reports earned higher trust and severity for stopping and asking (notes 02, 01).

## Pitfalls seen in the sources
- A lone DNS lookup with no HTTP request is common and may be a safety scanner, not your fetch (PortSwigger note; general). Chase the HTTP hit.
- Trailing newlines/CRLF in helper files silently break FFmpeg and raw-HTTP tricks (note 07).
- Timing is noisy; automated internal scans were unreliable and sometimes needed manual retries (notes 02, 09).
- Common web frameworks on your redirector can interfere; a hand-written raw HTTP server was needed for the chunked-download redirect (note 18).
- Metadata servers block framing; use a top-level navigation/redirect, not an iframe, in screenshot services (note 20).
