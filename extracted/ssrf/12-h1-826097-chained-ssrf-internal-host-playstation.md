# Chained SSRF through image proxy into a PhantomJS banner service (PlayStation)

Source completeness: full

- **Vuln type / sub-variant:** Chained SSRF: a public image-fetch/convert endpoint reaches an internal host, which itself is a second SSRF that supports the file scheme, giving local image read. Response must be a valid image.
- **Target type:** Image conversion proxy (image GET parameter, format parameter), plus an internal banner-rendering service.
- **Recon signal:** An endpoint that fetches a remote image by URL and re-encodes it. Hostnames containing "mgmt" signal internal-only hosts; those returned 403 externally.
- **Root cause:** The proxy fetched arbitrary URLs including internal hosts with no destination restriction (only the response type limited exfil: it had to be an image, and file: did not work on the first hop). The internal banner service (PhantomJS-based, overlays text on a backplate image given by URL) accepted the file scheme for its backplate parameter and had no validation.
- **Exploitation steps:**
  1. Use the proxy's image parameter to fetch an image hosted on an internal "mgmt" host that returns 403 directly. It was served successfully through the proxy.
  2. Find an internal banner-generating host that composes an image from a backplate URL plus price/text parameters. Called directly it timed out from outside.
  3. Pass the banner service URL as the image parameter of the public proxy (so the proxy requests the internal service for you).
  4. Set the banner's backplate parameter to a local file path using the file scheme, pointing at a system image file.
  5. Flow: public proxy to internal banner service, which reads the local file, overlays data, and returns the image; the proxy returns it to the attacker.
- **Bypasses used:** alternate protocol: file:// on the second hop; chained SSRF via internal service (pivot through trusted internal host); possible markup injection into the renderer (the PoC has an odd type value and SVG output, not explained in source).
- **Impact:** Reading internal-only images and local image files from the internal host. Limited to files that render as images. Severity rated high.
- **Bounty:** $1,000
- **Lesson / pattern:** When a first SSRF is constrained (valid image only, no file scheme), look for internal services reachable through it that have their own URL-fetching features, and chain them.
- **Source:**
  - Title: SSRF chained to hit internal host leading to another SSRF which allows to read internal images.
  - Author: bugdiscloseguys
  - URL: https://hackerone.com/reports/826097
  - Date: 2021-03-30 (disclosed; submitted 2020-03-21)
  - Disclosed report: https://hackerone.com/reports/826097
  - Raw file: `sources/ssrf/12-h1-826097-chained-ssrf-internal-host-playstation.md`

Tags: image-proxy, chained-ssrf, file-protocol, phantomjs, internal-host, full-read
