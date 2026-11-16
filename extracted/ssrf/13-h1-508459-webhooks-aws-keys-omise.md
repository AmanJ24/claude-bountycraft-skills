# Webhook SSRF via 303 redirect leaks AWS role credentials (Omise)

Source completeness: full

- **Vuln type / sub-variant:** Full-read SSRF in webhook delivery, bypassing a "do not follow redirects" restriction with a specific redirect status code.
- **Target type:** Webhook (dashboard webhook endpoint with a "Recent Deliveries" view showing response bodies).
- **Recon signal:** Webhook delivery history exposes the response body, so any server-side fetch is fully readable. Redirects appeared not to be followed in general, which suggested a rule worth probing for gaps.
- **Root cause:** Redirects were blocked for common statuses but a 303 See Other was still followed, and the redirect target was not validated against internal ranges.
- **Exploitation steps:**
  1. Host a script on a public server that replies with a 303 and a Location header pointing at the metadata service's IAM credentials path (169.254.169.254, role path).
  2. Point the dashboard webhook endpoint at that script.
  3. Trigger an event, e.g. make a harmless API call such as adding a user.
  4. Open the webhook's Recent Deliveries, see a 200 status for the delivery, and read the stored response body, which contained the role's temporary credentials.
- **Bypasses used:** redirect via attacker-controlled 303 (the one status code the redirect restriction missed).
- **Impact:** Disclosure of AWS credentials for the EC2 instance role from the metadata service, enabling access to the AWS environment and internal enumeration. Rated high (7.1).
- **Bounty:** not stated in source
- **Lesson / pattern:** If redirects seem blocked, try every redirect code (301, 302, 303, 307, 308) separately; blocklists on status codes are often incomplete.
- **Source:**
  - Title: SSRF in webhooks leads to AWS private keys disclosure
  - Author: honoki
  - URL: https://hackerone.com/reports/508459
  - Date: 2019-06-28 (disclosed; submitted 2019-03-12)
  - Disclosed report: https://hackerone.com/reports/508459
  - Raw file: `sources/ssrf/13-h1-508459-webhooks-aws-keys-omise.md`

Tags: webhook, redirect, cloud-metadata, aws, full-read, filter-bypass
