# Webhook SSRF via redirect plus IPv4-mapped IPv6 address (HackerOne org webhooks)

Source completeness: full (H1 report plus author's Medium retelling; screenshots not available as text)

- **Vuln type / sub-variant:** Full-read-ish SSRF in a webhook "Test request" feature. The URL filter was bypassed by an attacker-controlled redirect pointing at an IPv4-mapped IPv6 literal.
- **Target type:** Webhook (organization program settings, test-delivery button with a viewable delivery log).
- **Recon signal:** A webhook feature takes a user-supplied URL, makes a server-side request, and shows delivery logs. It had visible filtering: internal IPv4 in dotted and decimal forms was rejected, but redirects were followed. The mix of "filter exists" and "redirects allowed" suggested checking whether the destination was re-validated.
- **Root cause:** The block-list was incomplete. It covered IPv4 representations but not IPv6 addresses that map onto IPv4 (the ::ffff:a.b.c.d family). Also, redirects were followed without the target of each hop being validated against the same rules, and (per the Medium post) octal forms were also reachable after a redirect.
- **Exploitation steps:**
  1. Host a small script on a public server that answers every request with a Location header pointing at an IPv4-mapped IPv6 literal for the link-local metadata address (the hex-compressed form, roughly http://[::ffff:a9fe:a9fe]).
  2. Create a webhook in the organization's program settings that points at that public script URL.
  3. Edit the webhook and press "Test request".
  4. Read the webhook delivery log: the response headers showed a Server value typical of the AWS instance metadata service, proving the server fetched the metadata address.
  5. (Medium retelling) Escalate to internal port probing by making the redirector take a target URL parameter, e.g. http://[::ffff:7f00:1]:PORT, and iterating ports with Burp Intruder style substitution. Timing was too inconsistent for automated scans, so ports were checked manually and a handful of internal services were found (ports 5000, 5012, 8090, 8126, 24220).
- **Bypasses used:** redirect via attacker-controlled 302 (no re-validation of redirect target); IPv6 address form (IPv4-mapped, hex-compressed); octal IP encoding also noted as working post-redirect.
- **Impact:** Reached the cloud metadata service and several internal ports from the webhook sender; response headers (and logs) visible to the reporter. Severity was downgraded to medium (4.9) by the program. Whether metadata credentials were actually read is not stated in source.
- **Bounty:** $2,500
- **Lesson / pattern:** A filter that is "complete" for IPv4 is not complete: test IPv4-mapped IPv6, and always test redirect targets separately because they are often validated differently (or not at all).
- **Source:**
  - Title: Server Side Request Forgery (SSRF) in webhook functionality (H1 report); A Real SSRF Story from HackerOne (Featuring IPv6 + Redirects) (Medium)
  - Author: madara_ (H1 report); Red Darkin with madara_ (Medium)
  - URL: https://hackerone.com/reports/2301565 ; https://medium.com/@red_darkin/a-real-ssrf-story-from-hackerone-featuring-ipv6-redirects-9aa5e2ad8c2e
  - Date: 2024-01-30 (disclosed; reported 2024-01-02) ; Medium 2026-04-15
  - Disclosed report: https://hackerone.com/reports/2301565
  - Raw file: `sources/ssrf/09-h1-2301565-webhook-ssrf.md` and `sources/ssrf/24-medium-red-darkin-ipv6-redirect-webhook.md`

Tags: webhook, redirect, ipv6, cloud-metadata, filter-bypass, port-scan
