# Custom integration webhook SSRF exposes AWS metadata (Dynatrace)

Source completeness: partial (team summary and timeline only; no technical details in public text)

- **Vuln type / sub-variant:** SSRF in a user-configured webhook integration leading to cloud metadata access. Read capability not detailed.
- **Target type:** Webhook (custom integration that sends updates to a customer-chosen endpoint).
- **Recon signal:** Not stated in source. The feature itself (customer-defined outbound endpoint) is the obvious signal.
- **Root cause:** Not stated in source. The summary only says the webhook setup could be made to obtain AWS instance metadata.
- **Exploitation steps:** Not stated in source. Presumably configuring the webhook with a metadata address, but this is not confirmed by the text.
- **Bypasses used:** not stated in source.
- **Impact:** AWS metadata obtained through the webhook. Triager rated it critical (10.0). The vendor found no evidence of abuse and fixed it in a managed release.
- **Bounty:** $1,500 (awarded the day after submission)
- **Lesson / pattern:** Webhook and "custom integration" endpoints are a recurring metadata-reachability spot; the same feature class should be tested in any SaaS with outbound notifications.
- **Source:**
  - Title: SSRF in the Custom Integration Webhook discloses AWS metadata
  - Author: mohammad_obaid
  - URL: https://hackerone.com/reports/643278
  - Date: 2020-03-23 (disclosed; submitted 2019-07-15)
  - Disclosed report: https://hackerone.com/reports/643278
  - Raw file: `sources/ssrf/11-h1-643278-custom-integration-webhook-aws.md`

Tags: webhook, cloud-metadata, aws, saas-integration
