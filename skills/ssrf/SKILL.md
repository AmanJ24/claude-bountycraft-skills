---
name: ssrf
description: Use when assessing a web app or API feature that makes the server fetch a URL, address, or resource you can influence (webhooks, "test connection" buttons, link previews/unfurls, import-from-URL, avatar/image fetchers, PDF and screenshot generators, document and video processors). Helps recognize the surface, test it safely, bypass SSRF filters, avoid false positives, and write up impact. Research/methodology only: it does not send requests to live targets.
---

# SSRF (Server-Side Request Forgery)

Synthesized from 24 real bug bounty findings and research writeups (HackerOne disclosed reports, PortSwigger Research, a NahamSec talk, practitioner writeups). Every pattern below traces to `sources.md`. Bounties in the set ranged from $150 to $25,000; the biggest payouts came from reaching cloud metadata or chaining into infrastructure.

**Scope rule:** this skill produces methodology and guidance. It does not fire requests at live targets, and it never touches real cloud-metadata or internal infrastructure without explicit, scoped human authorization (see `../../CLAUDE.md`, Phase 2/3). "Confirming" SSRF can mean reaching real internal systems, which crosses into unauthorized access if uncontrolled.

## 1. Recognize it

SSRF is present whenever the **server** (not your browser) makes a request to a destination you can influence. The tell is almost always a feature that is *supposed* to fetch something for you.

Core signals, in rough order of how often they paid out in this set:
- A field or parameter holds a **URL, hostname, or IP** that the backend then retrieves: `url`, `uri`, `src`, `dest`, `target`, `source`, `callback`, `webhook`, `feed`, `endpoint`, `host`, `redirect`, `return`, `data`, `path`, `domain`, `shop_url`, `resturl`.
- A feature **named for fetching**: webhook, "test connection"/"send test event", link preview/unfurl, import from URL, "add image by URL", avatar/logo from URL, remote backup, feed reader, site auditor.
- A feature that **transforms user input into a document or image on the server**: HTML-to-PDF export (invoices, reports, checklists), screenshot/thumbnail service, SVG/image processing, video transcode. These carry SSRF even when there is no URL field, because injected markup inside the renderer makes the fetch.
- An **outbound callback you can observe**: delivery logs, "recent deliveries", preview titles, error text, or a generated file that embeds the fetched response.

A useful first test is a single out-of-band callback (your own Collaborator/webhook.site/interactsh host). A DNS lookup or HTTP hit from the target's infrastructure confirms server-side fetch. See `references/testing-methodology.md`.

## 2. Where it hides

The feature classes that actually produced findings here. Full list with example reports in `references/attack-surfaces.md`.

- **Webhooks and custom integrations** (Omise, Dynatrace, HackerOne, Slack slash commands). Delivery logs often make it full-read. Highest-frequency surface in the set.
- **PDF / document generators** that render HTML server-side (DoD checklist tool, HackerOne analytics export, the NahamSec talk, PortSwigger PDF research). Injected `<iframe>`/`<script>`/`<img>` reaches internal URLs and metadata.
- **Headless-browser render / screenshot services** (Shopify Exchange store screenshotter, h1-CTF headless Chrome). The page you control *is* the SSRF vehicle; can reach localhost debug ports.
- **Link preview / URL unfurl** (Matrix/Reddit chat preview). Even a leaked page title enumerates internal services.
- **Import-from-URL** (GitLab project import, Lark Docs import, Concrete CMS remote file, Vimeo video import). Importers often return converted content, giving full read.
- **"Test connection" / endpoint-validation buttons** in admin consoles (Apache Airflow). Server fetches an attacker-chosen host and echoes the response.
- **Image / media processors**: SVG (`xlink:href`, UNC/SMB paths) through ImageMagick (Shopify, Rockstar), FFmpeg HLS playlists in video upload (TikTok), image-fetch proxies (PlayStation).
- **Third-party tooling left exposed**: self-hosted Sentry with source-scraping on (HackerOne). Fingerprint from CSP report-uri and headers.

## 3. Test methodology

Short version. Full step-by-step in `references/testing-methodology.md`.

1. **Map the fetch.** Find every parameter and feature that could cause a server-side request. Check error paths and JSON save calls, not just happy-path inputs (the $25k HackerOne bug was a reflected value inside a "template not found" error).
2. **Confirm server-side fetch** with one OOB callback. Note the callback's User-Agent: it fingerprints the engine (FFmpeg/Lavf, wkhtmltopdf, Prince, headless Chrome) and tells you which engine-specific tricks apply.
3. **Classify what comes back:** full response visible (read body/delivery log/generated file), partial (title/headers/error only), or blind (callback only, no body). This decides escalation and severity.
4. **Probe reach** without touching real internal infra unless authorized: your own hosts and ports first to prove arbitrary destination and port reach.
5. **If a filter blocks internal addresses,** work the bypass ladder in `references/bypasses.md` (redirects, IPv6, IP-notation encodings, DNS rebinding, resolver quirks, alternate protocols).
6. **Escalate impact** per `references/exploitation-and-escalation.md`: cloud metadata (AWS/GCP/Azure, including legacy no-header paths), internal port scan, local file read (`file://`, FFmpeg, UNC/SMB), chaining through a second internal SSRF, reaching localhost admin/debug ports.
7. **Stop at proof.** Demonstrate access (one credential, one internal service name, one file's first line) and report. Do not pivot into the environment.

## 4. Known bypass / evasion patterns

When an obvious payload (`http://127.0.0.1`, `http://169.254.169.254`) is rejected, a filter exists and is usually a blocklist on the input *text*, not the final resolved destination. Summary; full catalogue with payload shapes in `references/bypasses.md`.

- **Redirects.** Point the feature at your server; reply with a 3xx `Location` to the internal target. Test *every* status (301/302/303/307/308): Omise followed only 303. Redirect targets are usually re-validated less strictly, or not at all.
- **IPv6 forms.** `[::]`, `[::1]`, IPv4-mapped `[::ffff:169.254.169.254]` / `[::ffff:a9fe:a9fe]` when only IPv4 literals are blocked (Slack, HackerOne webhook).
- **IP-notation encodings.** Octal (`0177.0.0.1`), hex (`0x7f.0.0.1`), dword (`2130706433`), zero-padded, mixed. Defeat string/regex comparisons (GitLab, WooCommerce, Nextcloud).
- **DNS rebinding.** Hostname resolves public at check time, internal at fetch time (check/use gap). Beats filters that validate the resolved IP once (Concrete CMS).
- **Resolver quirks.** Ruby `Resolv.getaddresses` returns `[]` for some shorthand IPs, so a blocklist finds "no private IP" while the OS still connects to loopback (GitLab + HackerOne Integrations, CVE-2017-0904). Validate with the *same* resolver the HTTP client uses.
- **Unicode / normalization.** Circled digits and fullwidth/ideographic dots that are "not an IP" to the validator but normalize to one later (Nextcloud).
- **Allowlist weaknesses.** Open redirect on an allowed domain; weak matching (allowed name as a subdomain/path of your domain).
- **Alternate protocols.** `file://`, `gopher://`, `ssh://`, UNC/SMB (`\\host\share` leaks NTLM hash), `dict://`. Try when `http` is filtered or for local file read.
- **Markup injection into renderers** (no URL field needed): `<iframe>`/`<script>`/`<img>`/SVG `xlink:href` in PDF/screenshot/image pipelines; CSS/style breakout when only styling is accepted; PDF annotation parenthesis breakout.
- **Second-request gaps.** Multi-step fetchers (chunked/range downloads, resumes, retries) may skip on request two the validation applied to request one (Vimeo).

## 5. Common false positives

- **Client-side only requests.** If the request comes from your browser (visible in devtools, uses your cookies/IP), it is not SSRF. Confirm the fetch originates from the target's infrastructure (callback source IP/User-Agent).
- **A DNS lookup with no HTTP request.** Common and expected when testing; a lone DNS hit may be a security scanner, link-safety check, or antivirus on the target side, not a fetch you can escalate. Look for the actual HTTP request.
- **Fully external-only fetch with no internal reach and no interesting response.** An "open" SSRF that can only hit public sites and returns nothing may be low/informational unless you can show internal reach, a readable response, or a dangerous protocol.
- **Intended webhooks/integrations with proper egress controls.** If the feature is meant to call arbitrary external URLs and internal ranges are genuinely blocked (post-redirect, post-resolution), a plain callback is by-design, not a bug. The finding is the *bypass*.
- **Vendor WAF / CDN interactions** that echo your payload back but never actually fetch it.
- **Self-inflicted reflection** in a generated PDF that renders markup but cannot make network requests (some HTML-to-PDF engines disable network/script): confirm an actual outbound hit before claiming SSRF.

## 6. Reporting

What made reports in this set land well (detail and a template in `references/reporting.md`):
- **Lead with demonstrated impact, not the callback.** "Reached AWS metadata and read the instance role credentials" beats "server made a request to my host." The $25k and $5k findings showed a credential or clear internal reach.
- **Show the class of response you got** (full/partial/blind) and exactly what you read: one credential, one internal service banner, the first line of a file. Redact the secret value itself.
- **State the escalation you did NOT do, and why.** Several high ratings came with the researcher explicitly stopping at proof and asking before going further (Matrix, HackerOne analytics). This raises trust and severity, not lowers it.
- **Name the root cause precisely** (incomplete redirect handling, blocklist on text, resolver mismatch) so triage can reproduce and fix. Give the exact bypass string.
- **Tie to concrete risk:** metadata credentials to cloud account access, internal port scan to attack surface, file read to secrets, chained SSRF to infrastructure. CVSS tends to land high when cloud credentials or internal RCE paths are shown.

## 7. Sources

See `sources.md` for the full list (title, author, URL, disclosed report, bounty) and `../../extracted/ssrf/` for the per-writeup notes each claim is drawn from. Deeper references:
- `references/attack-surfaces.md`: every feature class with example reports
- `references/testing-methodology.md`: step-by-step probing
- `references/bypasses.md`: full filter-bypass catalogue with payload shapes
- `references/exploitation-and-escalation.md`: metadata (AWS/GCP/Azure/ECS/k8s), file read, chaining, gopher-to-Redis and other protocols
- `references/reporting.md`: write-up template

**Related skills (as this toolkit grows):** SSRF commonly arrives through, or escalates into, another class. Renderer/markup SSRF overlaps with file-upload and XXE sinks; a `file://` read of a k8s service-account token turns SSRF into cluster API access; a gopher payload to internal Redis is an RCE path. When a sibling skill exists for the second half, use it.
