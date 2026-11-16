# HackerOne: organization_id on /bugs.json search leaks private reports across programs

- **Vuln type / sub-variant:** Cross-tenant IDOR via a numeric organization id in a search/list POST body (object-level read, not a single-record fetch).
- **Target type:** Internal JSON search/listing API behind the main dashboard (report list with filters, paging).
- **Recon signal:** Not stated in source beyond the request shape. The listing endpoint takes an `organization_id` parameter chosen by the client, and the policy page exposes an example org id, which gave a ready value to try.
- **Root cause:** The endpoint filtered reports by the supplied `organization_id` without checking that the caller was a member of, or had report access to, that organization. Authorization was applied to the session, but the tenant scoping came straight from the body.
- **Exploitation steps:**
  1. Log in with an ordinary account and replay the listing POST to `/bugs.json`.
  2. Set `organization_id` to another program's id (an id from a public policy page worked) and add a one-character `text_query`, with the broad `substates[]` list so all states match.
  3. Later refinement: drop the text search, set `limit=1000` and `page=1` to page everything for an org, and add the `editing` substate to also see drafts.
  4. Read the response metadata for private reports.
- **Bypasses used:** none. Only parameter tuning (limit/page, extra substates) to widen results. Reporter said private programs were also affected.
- **Impact:** Disclosure of private report metadata across all organizations: title, url, id, state, severity, timestamps and reporter name. Reporter stopped once cross-org leakage was confirmed. CVSS 9.3 critical, confidentiality high, integrity none.
- **Bounty:** Initial reward amount hidden; $500 awarded for the retest.
- **Lesson / pattern:** List and search endpoints that accept a tenant or org id from the client are IDORs too. Swap the org id for one you do not belong to and look at what comes back.
- **Source:**
  - Title: Insecure Direct Object Reference (IDOR) Allows Viewing Private Report Details via /bugs.json Endpoint
  - Author: bate5a
  - URL: https://hackerone.com/reports/2487889
  - Date: reported 2024-05-02, disclosed 2024-05-23
  - Disclosed report: https://hackerone.com/reports/2487889
  - Raw file: `sources/idor/02-h1-2487889-hackerone-bugs-json-idor.md`

Tags: cross-tenant, numeric-id, api, pii-disclosure, enumeration
