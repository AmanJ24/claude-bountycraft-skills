# Half-blind SSRF in Kubernetes controller manager via StorageClass, upgradable to full SSRF

Source completeness: full (some screenshots and the PoC archive are not in the text)

- **Vuln type / sub-variant:** Half-blind SSRF from a managed-cloud control plane, upgraded to full-response SSRF through a Go HTTP client request-splitting bug on old versions
- **Target type:** Managed Kubernetes service (vendor-run control plane), triggered through a customer-defined StorageClass and PersistentVolumeClaim
- **Recon signal:** Reading the Go source of in-tree volume provisioners (GlusterFS/heketi client, also ScaleIO and StorageOS) showed that a customer-controlled REST URL parameter was used by the controller manager, which runs inside the vendor's internal network rather than the customer's.
- **Root cause:** The provisioner built its HTTP request from the user-supplied resturl value, and the controller manager that issued it sat in the provider's trusted perimeter. A fixed suffix path was appended, which a fragment character neutralized. On old Go versions the client also failed to reject CR/LF in the URL.
- **Exploitation steps:**
  1. Create a managed cluster, then apply a StorageClass using the GlusterFS provisioner with resturl set to an attacker host plus a trailing fragment marker, and a PVC that uses it.
  2. Observe the controller manager's request arriving at the attacker server (confirms half-blind SSRF).
  3. Make the attacker URL answer with a 302 to an internal target so the POST becomes a GET; read results via PVC description or cluster events when the response was JSON or an error.
  4. Automate scanning of an internal range with parallel workers that recreate StorageClass and PVC per target (slow because each probe needs a new object).
  5. On clusters with old Go (before 1.12) where the customer could read controller-manager logs, inject CRLF plus a second request into resturl to craft full HTTP requests and read full responses from the logs.
- **Bypasses used:** URL fragment to cut the appended path; redirect via attacker-controlled 302 (POST to GET conversion); CRLF / request smuggling in the Go HTTP client (old versions only).
- **Impact:** Internal network scanning of the provider's master VPC; reading cloud metadata (credential theft path), kubelet, etcd reachability; potential lateral movement, DoS of masters, and cross-customer impact. Severity varied per vendor.
- **Bounty:** $5,000
- **Lesson / pattern:** Features where a customer supplies a URL that a vendor-side control plane component fetches (storage backends, webhooks, registries) are SSRF into the provider network; combine 302 redirects with output side channels such as events or logs.
- **Source:**
  - Title: Half-Blind SSRF found in kube/cloud-controller-manager can be upgraded to complete SSRF (fully crafted HTTP requests) in vendor managed k8s service
  - Author: reeverzax (with hach)
  - URL: https://hackerone.com/reports/776017
  - Date: 2020-10-30 (reported 2020-01-15)
  - Disclosed report: https://hackerone.com/reports/776017
  - Raw file: `sources/ssrf/03-h1-776017-kube-cloud-controller-manager-ssrf.md`

Tags: kubernetes, half-blind, redirect, crlf-smuggling, cloud-metadata, managed-service
