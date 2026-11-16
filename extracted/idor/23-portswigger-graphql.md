# PortSwigger Web Security Academy: GraphQL vulnerabilities, IDOR via unsanitized arguments (reference)

- **Vuln type / sub-variant:** GraphQL IDOR from unsanitized object arguments, plus supporting recon: endpoint discovery, introspection and its bypasses, suggestions, alias-based rate limit bypass, CSRF over GraphQL. Only the first is IDOR proper; the rest enable it.
- **Target type:** GraphQL APIs (single endpoint for all operations).
- **Recon signal:** A list query returns objects with sequential IDs and a gap (a missing ID such as a delisted item), suggesting hidden objects reachable by direct query.
- **Root cause:** Resolvers accept an ID argument and return the object without checking access, so objects excluded from list views are still fetchable by ID. Introspection left on, suggestions left on and per-HTTP-request rate limiting make discovery and brute force easier.
- **Exploitation steps:** (method taught)
  1. Locate the endpoint: send the universal __typename query; try common paths (/graphql, /api, /api/graphql, /graphql/api, /graphql/graphql, then with /v1); if POST fails, try GET or form-encoded.
  2. Observe normal traffic in the proxy history to learn operations.
  3. Read list results, infer sequential IDs and find gaps, then query the single-object operation with the missing ID to retrieve unlisted or private objects.
  4. Map the schema: probe with a minimal __schema query, then run a full introspection query (drop onOperation, onFragment, onField directives if rejected), optionally visualize it.
  5. If introspection is disabled, use error-message suggestions (tool named: Clairvoyance) to rebuild the schema.
  6. Use schema to find every operation taking an ID and test each.
- **Bypasses used:** Introspection filter bypass by inserting characters GraphQL ignores (space, newline, comma) after the __schema keyword to defeat naive regex; retrying introspection over GET or form-encoded POST when only JSON POST is blocked; aliases to pack many guesses (for example discount codes) into one HTTP request and dodge per-request rate limiting; form-encoded or GET requests enabling CSRF when content type is not validated.
- **Impact:** Disclosure of hidden or private objects; schema disclosure; brute force of codes; CSRF-driven actions as the victim. Labs listed: accessing private GraphQL posts, accidental exposure of private fields, finding a hidden endpoint, bypassing brute force protections, CSRF over GraphQL.
- **Bounty:** n/a
- **Lesson / pattern:** In GraphQL, list endpoints leak ID structure; query the gaps directly, and use introspection or suggestions to find every ID-taking operation.
- **Source:**
  - Title: GraphQL API vulnerabilities
  - Author: PortSwigger
  - URL: https://portswigger.net/web-security/graphql
  - Date: not stated in source
  - Disclosed report: none
  - Raw file: `sources/idor/23-portswigger-graphql.md`

Tags: graphql, methodology, enumeration, bypass, numeric-id
