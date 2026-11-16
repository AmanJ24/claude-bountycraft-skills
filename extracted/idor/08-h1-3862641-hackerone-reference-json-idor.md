# HackerOne: reference.json error response leaks private report details

Source completeness: partial (only HackerOne's public summary is available; the original report body and most timeline comments are hidden)

- **Vuln type / sub-variant:** Numeric report-id IDOR via information leak in an access-denied error body (authorization enforced on status code but not on payload).
- **Target type:** Internal JSON endpoint `/reports/{id}/reference.json` reachable by low-privilege VDP-signup accounts.
- **Recon signal:** Not stated in source. The summary only says low-privilege accounts could call the endpoint with arbitrary report ids.
- **Root cause:** The endpoint correctly returned HTTP 422 "no access", but the error object still embedded the report's subject details under `context.subject`. The check blocked the response semantics while the error serializer disclosed data about the target object.
- **Exploitation steps:**
  1. Sign up for a VDP account.
  2. Pick a target report id (the program's own approved test reports for PoC).
  3. POST an empty JSON body `{}` to the endpoint with that id.
  4. Ignore the 422 and read the report details in `context.subject`.
- **Bypasses used:** none beyond reading the error body.
- **Impact:** Disclosure of certain private report details belonging to other programs. HackerOne states no evidence of misuse. Rated critical.
- **Bounty:** $15,000.
- **Lesson / pattern:** Do not stop at a 403/422. Read the full body of denied responses, since error serializers often include the object that was just refused.
- **Source:**
  - Title: IDOR allows user to access report details via reference.json endpoint
  - Author: sm41ldrag0n79_mbbank
  - URL: https://hackerone.com/reports/3862641
  - Date: reported 2026-07-14, disclosed 2026-10-01
  - Disclosed report: https://hackerone.com/reports/3862641
  - Raw file: `sources/idor/08-h1-3862641-hackerone-reference-json-idor.md`

Tags: numeric-id, api, pii-disclosure, cross-tenant
