# DNS rebinding bypass of metadata-service filter in remote file upload (Concrete CMS)

Source completeness: partial (very short report; screenshots not available as text, no payload details)

- **Vuln type / sub-variant:** Full-read SSRF with SSRF-mitigation bypass via DNS rebinding.
- **Target type:** File upload from a remote URL (import file from remote server).
- **Recon signal:** The upload feature fetches a remote file, and the app already had mitigations specifically against reaching the AWS instance metadata service, so the check was worth attacking.
- **Root cause:** The hostname was validated once but resolved again when the request was made (check/use gap), so a hostname that first resolves to a public IP and later to the metadata IP got through. Exact implementation not stated in source.
- **Exploitation steps:**
  1. Use a public DNS rebinding service to produce a hostname that alternates between a harmless address and the metadata address.
  2. Give that hostname to the remote file upload feature on an instance running in AWS.
  3. The fetched content was the IAM key material from the metadata service (exact request path not stated in source).
- **Bypasses used:** DNS rebinding (via a public rebinding service).
- **Impact:** Retrieval of the AWS IAM keys the application runs under. Rated low (3.5) by the program.
- **Bounty:** not stated in source
- **Lesson / pattern:** An IP filter on the resolved name is not enough if the connection re-resolves; test rebinding whenever a fetch feature explicitly blocks metadata IPs.
- **Source:**
  - Title: SSRF mitigation bypass using DNS Rebind attack
  - Author: adrian_t
  - URL: https://hackerone.com/reports/1369312
  - Date: 2022-11-25 (disclosed; submitted 2021-10-13)
  - Disclosed report: https://hackerone.com/reports/1369312
  - Raw file: `sources/ssrf/15-h1-1369312-dns-rebind-bypass-concretecms.md`

Tags: dns-rebinding, import-from-url, cloud-metadata, aws, filter-bypass, full-read
