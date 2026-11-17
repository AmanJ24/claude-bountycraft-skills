# Editors create project secrets via API while UI hides the Manage section

- **Vuln type / sub-variant:** UI-vs-API authorization mismatch (role and plan restriction enforced only in the front end), vertical privilege escalation
- **Target type:** SaaS developer platform with Owner, Editor, and Viewer roles and free-tier plan limits; secrets management REST API
- **Recon signal:** The UI hid the Manage (secrets) section for Editors and for free-tier projects, which implied a rule worth checking directly against the API.
- **Root cause:** The secrets endpoint did not enforce role or plan checks server-side; it accepted any authenticated project member. Restriction existed only as hidden UI.
- **Exploitation steps:**
  1. Join a free-tier project as an Editor and confirm the Manage section is not shown.
  2. Intercept normal traffic with a proxy to learn the API shape.
  3. Send a hand-crafted POST to the secrets endpoint under the project namespace with an arbitrary secret name and value.
  4. Server returns success and the secret exists in the project.
- **Bypasses used:** Skipping the UI entirely and calling the backend directly with a crafted request.
- **Impact:** Editor performs Owner-only actions, free-tier limit bypassed, risk of injecting secrets that influence deployments or pipelines. Rated High (CVSS 8.3).
- **Bounty:** $1,500 (submitted July 5, 2025, fixed and paid by July 10).
- **Lesson / pattern:** For every feature hidden from a role or plan in the UI, replay the underlying API call as that low-privilege account. Build a role-by-endpoint matrix from the highest role's traffic and replay it with the lower ones.
- **Source:**
  - Title: $1,500 Bounty: Business Logic Flaw in Project Secrets Management
  - Author: Abhi Sharma (@a13h1)
  - URL: https://infosecwriteups.com/1-500-bounty-business-logic-flaw-in-project-secrets-management-4b985175071e
  - Date: 2025-09-19
  - Disclosed report: none
  - Raw file: `sources/business-logic/25-infosecwriteups-secrets-mgmt-ui-api-mismatch.md`

Tags: ui-vs-api, privilege-escalation, privilege-logic, client-trust, missing-server-validation, premium-bypass
