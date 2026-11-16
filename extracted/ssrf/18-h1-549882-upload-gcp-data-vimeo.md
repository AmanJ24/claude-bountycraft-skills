# Vimeo upload-from-URL SSRF via partial-content redirect to GCP metadata

- **Vuln type / sub-variant:** Full-read SSRF via redirect during chunked (Range-based) download; escalated to cloud metadata token theft.
- **Target type:** Video import from cloud storage (Google Drive picker): the browser sends the file URL plus the user's Google auth header, and Vimeo's backend fetches it.
- **Recon signal:** The import request carried a fetch URL and a forwarded Authorization header, so the backend was clearly downloading attacker-influenced URLs. Pointing it at a hunter-owned server and sniffing the traffic revealed Range and Content-Range headers, meaning the backend downloads big files in pieces and stitches them together.
- **Root cause:** The downloader trusts the Content-Length it was told about first. If the server returns fewer bytes than promised, the client keeps requesting the remainder, and on that follow-up request it follows redirects without re-validating the destination. The response body from the redirect target is stored as part of the "video" and the user can download the original file, so the forged response is readable.
- **Exploitation steps:**
  1. Run a custom HTTP server and submit its URL via the Google Drive import path.
  2. Observe the size probe and the ranged requests; answer with a declared large length but deliver only a small first chunk.
  3. Watch the backend come back asking for the remaining range.
  4. On the second request, answer with a redirect to an internal address instead of data. A raw hand-written HTTP server was needed because common web frameworks interfered.
  5. Confirm the redirect target's response was stored, then download the original uploaded file to read it.
  6. Determine the host was a Google Cloud instance and redirect to the metadata service token path (v1beta1 service-accounts default token), obtaining a Compute Engine API access token.
- **Bypasses used:** Redirect via attacker-controlled response mid-download (second-stage request after a short first chunk), plus the legacy v1beta1 metadata path that did not need the Metadata-Flavor header. No explicit filter bypass was needed.
- **Impact:** Critical on H1 (score 9.0). Read the instance service account access token from GCP metadata. The H1 title also mentions SSH keys and other internal cloud data.
- **Bounty:** not stated in source (author says it was paid within two days, in two installments; amount not given).
- **Lesson / pattern:** Any multi-request fetcher (chunked downloads, resumes, retries) has a second request that may skip the validation applied to the first; make the server misbehave (short body, odd status, redirect) on request two or later and watch what it does.
- **Source:**
  - Title: SSRF leaking internal google cloud data through upload function (H1 report) and "Vimeo upload function SSRF" (author writeup, merged)
  - Author: dphoeniixx (Sayed Abdelhafiz)
  - URL: https://hackerone.com/reports/549882 ; https://medium.com/@dPhoeniixx/vimeo-upload-function-ssrf-7466d8630437
  - Date: 2019-12-13 (disclosed); H1 report submitted 2019-04-28
  - Disclosed report: https://hackerone.com/reports/549882
  - Raw file: `sources/ssrf/18-h1-549882-upload-gcp-data-vimeo.md` ; `sources/ssrf/29-medium-vimeo-upload-ssrf-gcp.md`

Tags: import-from-url, redirect, cloud-metadata, full-read, gcp, range-requests
