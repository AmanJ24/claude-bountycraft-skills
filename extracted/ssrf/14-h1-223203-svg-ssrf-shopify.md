# SVG image upload SSRF with picture-presence oracle (Shopify)

Source completeness: full (the report is an extended probe; no cloud-metadata access shown)

- **Vuln type / sub-variant:** Blind SSRF through SVG processing of an uploaded product image, with a side-channel oracle for local file existence. A regression of an earlier fixed issue.
- **Target type:** Image upload (product images in the store admin).
- **Recon signal:** The reporter suspected a regression of a prior SVG issue and tested whether the file was processed before the file type check ran.
- **Root cause:** The upload was passed to the image processing library before validation, so an SVG disguised as an image was parsed and its external references fetched. The check on type came too late.
- **Exploitation steps:**
  1. Send the product image upload request with a filename ending in .png but declared content type image/svg+xml and an SVG body containing an image element whose xlink:href points at an attacker server.
  2. The server answered 422 "corrupt image", yet the attacker server still got a DNS lookup and an HTTP GET.
  3. Probe: only http and ftp schemes produced outbound interaction. All TCP ports were reachable outbound except one (113).
  4. Entity behavior: static entities were accepted (so a billion-laughs style DoS is plausible, out of scope), but SYSTEM entities and external DTDs were rejected, so no XXE.
  5. Oracle: place two image references in one SVG. The second is only fetched if the first resolves to a valid image. Using a local path as the first reference tells whether that image file exists on the server, which allowed fingerprinting installed library docs/examples.
- **Bypasses used:** content-type and extension mismatch to skip upload validation (disguised file type); HTML/XML injection into renderer (SVG xlink:href); no network filter evasion needed.
- **Impact:** Outbound HTTP/FTP from the server to arbitrary hosts and ports (port-reachability scan of the attacker's own host), plus a file-existence oracle limited to image files, enabling library version fingerprinting. No data read. Rated low.
- **Bounty:** $500
- **Lesson / pattern:** Upload validators that run after processing are no validators; for image uploads, always test SVG with external href even when the response is an error, and use a second reference for chained-fetch oracles.
- **Source:**
  - Title: SVG Server Side Request Forgery (SSRF)
  - Author: floyd
  - URL: https://hackerone.com/reports/223203
  - Date: 2017-09-22 (disclosed; submitted 2017-04-23)
  - Disclosed report: https://hackerone.com/reports/223203
  - Raw file: `sources/ssrf/14-h1-223203-svg-ssrf-shopify.md`

Tags: svg-upload, blind, file-oracle, image-processing, content-type-mismatch, oob-callback
