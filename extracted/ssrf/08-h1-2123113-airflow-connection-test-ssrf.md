# SSRF through Apache Airflow "test connection" feature (Slack connection type)

Source completeness: partial (short report, screenshot not in text, no step-by-step PoC)

- **Vuln type / sub-variant:** Non-blind SSRF through a connection-testing function in an admin UI
- **Target type:** Workflow platform feature where users define a connection and press "test" (Slack API connection type named)
- **Recon signal:** not stated in source beyond noticing the test connection function would call out to a configurable endpoint.
- **Root cause:** The test connection action made requests using user-configurable connection details with no restriction on destination, and returned the response text. The vendor advisory also notes the feature could be abused for denial of service by repeated tests.
- **Exploitation steps:**
  1. As a user with connection-edit permission, create or edit a Slack-type connection with a chosen target.
  2. Press the test connection button and read the plain response returned to the UI.
  3. Aim it at internal or metadata addresses (researcher states impact only; exact PoC not given).
- **Bypasses used:** none
- **Impact:** Readable responses from arbitrary targets (not blind), access to connection information, request flooding leading to DoS, possible hits on metadata endpoints. Requires an authenticated user with connection-edit rights. Fixed in Airflow 2.7.0. Severity medium.
- **Bounty:** $2,550
- **Lesson / pattern:** "Test connection", "validate endpoint" and "send test event" buttons in admin consoles make server-side requests with attacker-chosen hosts and often echo the result.
- **Source:**
  - Title: SSRF Vulnerability through Connection test feature
  - Author: sayoojbkumar
  - URL: https://hackerone.com/reports/2123113
  - Date: 2023-09-12 (reported 2023-08-25)
  - Disclosed report: https://hackerone.com/reports/2123113
  - Raw file: `sources/ssrf/08-h1-2123113-airflow-connection-test-ssrf.md`

Tags: test-connection, authenticated, full-read, admin-feature, dos-side-effect
