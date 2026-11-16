# Escape: GraphQL IDOR vulnerabilities and how to fix them (reference)

- **Vuln type / sub-variant:** GraphQL IDOR where a resolver fetches an object by ID argument without verifying the requester may see it. Conceptual explainer with a mitigation focus.
- **Target type:** GraphQL APIs; illustrative example is a blog post query by ID.
- **Recon signal:** A single-object query with an ID argument (e.g. fetching a post by ID) that returns author-owned content.
- **Root cause:** No authorization in the resolver between receiving the ID and returning the object, so flexible client-driven data access exposes whatever the ID points to.
- **Exploitation steps:** (method taught)
  1. Take a query that returns your own object by ID.
  2. Change the ID to another user's.
  3. Automate by looping over an ID range and collecting results into a file, which gives mass extraction when IDs are sequential.
- **Bypasses used:** none
- **Impact:** Illustrated as mass reading of others' content. Cites disclosed reports as real-world IDOR examples: PayPal #415081 (business owner adds secondary users from other accounts), Vimeo #42587 (password reset of any account, takeover), Shopify #56511 (expire other users' sessions by editing the request). Mentions regulatory exposure (GDPR, CCPA) and a healthcare bounty example.
- **Bounty:** not stated in source
- **Lesson / pattern:** Authorization must live in each resolver that takes an ID; unguessable IDs and rate limits help but do not replace per-object checks. Note that the article's sample fix compares user ID to the object ID, which is flawed logic (it should compare ownership of the object), so do not copy it as a model.
- **Source:**
  - Title: GraphQL IDOR Vulnerabilities: What They Are & How to Fix
  - Author: Tristan Kalos (Escape)
  - URL: https://escape.tech/blog/idor-in-graphql/
  - Date: 2023-09-14
  - Disclosed report: references H1 #415081, #42587, #56511
  - Raw file: `sources/idor/24-escape-idor-in-graphql.md`

Tags: graphql, methodology, enumeration, account-takeover, api
