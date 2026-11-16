# GraphQL lookup of deactivated embedded-submission-form UUIDs, found via Wayback

- **Vuln type / sub-variant:** GraphQL object lookup by UUID (query argument substitution) on an inactive/deleted object, with UUIDs harvested from web archives. Improper access control rather than classic cross-user ID swap.
- **Target type:** Bug bounty platform feature: embedded report-submission form, queried through the platform GraphQL endpoint with an `EmbeddedSubmissionPage` operation taking a `uuid` variable (also echoed as a URL query parameter).
- **Recon signal:** The feature had a known intentional exposure (active form links are meant to be public). The hunter noticed that a UUID for a form that had been disabled or deleted, which the UI treated as inactive, still returned data through the GraphQL query.
- **Root cause:** The resolver returns the form and its parent program data purely from the UUID and does not check whether the form is still active/published, so disabling a form did not revoke access. The only protection was UUID secrecy.
- **Exploitation steps:**
  1. Run the GraphQL query with an inactive UUID and see the owning program's details in the response (intro text for the report form, response-efficiency percentage, structured scope count, form customization fields, team handle and name).
  2. After the first report was closed as N/A (active links are public by design), argue the inactive-UUID case and cite a related disclosed report.
  3. When asked how an attacker would obtain UUIDs, show the recon: pull archived URLs for the platform domain with `waybackurls` and grep for the embedded submission path, yielding many UUIDs for private programs.
- **Bypasses used:** UUID unguessability bypassed via Wayback Machine URL harvesting; no payload tricks.
- **Impact:** Disclosure of information about private programs whose forms were deactivated (names, program metadata, scope counts). Rated Low/Medium (CVSS 3.7 to 4.4).
- **Bounty:** $2,500
- **Lesson / pattern:** When a "secret" UUID is the only control, prove both that revoked IDs still work and that real IDs can be harvested at scale (archives); a report becomes valid with a concrete attacker path.
- **Source:**
  - Title: IDOR on HackerOne Embedded Submission Form
  - Author: Japz Divino (pinoywhitehat)
  - URL: https://medium.com/pinoywhitehat/idor-on-hackerone-embedded-submission-form-9e59c6f044b3
  - Date: 2024-06-19
  - Disclosed report: related hacktivity #2483666 (not the finding itself; the report is in limited disclosure)
  - Raw file: `sources/idor/18-blog-embedded-submission-form-graphql-idor.md`

Tags: graphql, uuid, wayback-recon, enumeration, pii-disclosure
