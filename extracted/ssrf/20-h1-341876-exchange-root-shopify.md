# Shopify Exchange store screenshotter: SSRF to GCP metadata, kubelet creds, container root

- **Vuln type / sub-variant:** Full-read SSRF (via image output) in a headless-browser screenshot service, chained to cloud metadata and Kubernetes compromise.
- **Target type:** Marketplace feature that takes a screenshot of a seller's store (Create Listing page) using a server-side renderer.
- **Recon signal:** The service screenshots a store the attacker controls, and store themes allow custom template code with script. A page controlled by the hunter rendered by a server-side browser means arbitrary navigation from inside Shopify's network.
- **Root cause:** The screenshot renderer could be navigated to internal addresses by script in the store page, with no network egress restriction. The cluster nodes exposed GCP metadata, including legacy endpoints and kube-env, because metadata concealment was off.
- **Exploitation steps:**
  1. Create a partner store and edit the password page template so that its script redirects the window to the GCP metadata token URL (iframes were blocked by the metadata server's frame header, so a top-level redirect was used).
  2. Install the Exchange app and open Create Listing; wait for the store screenshot to be generated, then download the PNG and open it to read the token text.
  3. Work around plain-text responses not rendering by adding the alt=json query parameter to get JSON, which the screenshot did show. Leak the project SSH keys list, instance names and more.
  4. Use the legacy v1beta1 metadata path, which accepted requests without the required Metadata-Flavor header.
  5. Recursively dump instance attributes, find the kube-env attribute, and extract the kubelet client certificate, key and CA.
  6. Use these with kubectl to list and create pods; exec was denied, but describing pods exposed a secret name, from which a service account token was read.
  7. Use that token to exec into containers and get a root shell.
- **Bypasses used:** Missing-header bypass through legacy v1beta1 metadata endpoint; output-format trick (alt=json) to make the screenshot show data; top-level navigation instead of iframe because of framing restrictions; chained credential reuse. No IP filter bypass.
- **Impact:** Root in containers across one infrastructure subset (not Shopify core), kube-env secrets, SSH key list. The directly obtained GCP token had no Compute write access.
- **Bounty:** $25,000, awarded as a Shopify Core RCE-class finding (from team summary).
- **Lesson / pattern:** For screenshot or render services, make the page itself the SSRF vehicle, then see how far metadata goes: legacy endpoints, JSON output tricks and kube-env often turn a "medium" SSRF into infrastructure takeover.
- **Source:**
  - Title: SSRF in Exchange leads to ROOT access in all instances
  - Author: 0xacb
  - URL: https://hackerone.com/reports/341876
  - Date: 2018-05-23
  - Disclosed report: https://hackerone.com/reports/341876
  - Raw file: `sources/ssrf/20-h1-341876-exchange-root-shopify.md`

Tags: screenshot-service, headless-browser, cloud-metadata, gcp, kubernetes, full-read
