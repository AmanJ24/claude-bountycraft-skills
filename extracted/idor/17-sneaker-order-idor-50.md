# Unauthenticated order lookup by UUID in an online store

- **Vuln type / sub-variant:** UUID order ID IDOR (BOLA), read-only, no authentication required; ID obtained by consenting friend, not enumeration.
- **Target type:** New e-commerce store, REST endpoint `GET /api/orders/{order_uuid}` seen in browser dev tools. Store unnamed.
- **Recon signal:** After placing an order, the Network tab showed the order page loading from a UUID path. A random UUID gave 404, which left open whether the server checked ownership or the object simply did not exist, so the hunter tested against a real order from another user.
- **Root cause:** The handler only looks up the order by ID; it neither requires authentication nor compares the order owner to the requesting user.
- **Exploitation steps:**
  1. View own order and capture the GET request.
  2. Replace the UUID with a random one: 404 (inconclusive).
  3. Use a friend's real order UUID (shared with permission) while logged in as self: full order details returned.
  4. Clear cookies and repeat, then repeat with plain curl and no headers: identical 200 response.
- **Bypasses used:** None needed; the 404-on-random-ID behavior was misleading and had to be disproved with a known-valid foreign ID.
- **Impact:** Anyone holding an order UUID can read product, size, quantity, timestamp, delivery city and pickup point, and total, without login. Mass exploitation impractical due to UUIDs; rated medium by the author.
- **Bounty:** $50
- **Lesson / pattern:** A 404 on a made-up ID is not proof of authorization; test with a real second-account object and with auth removed.
- **Source:**
  - Title: How I Found an IDOR Vulnerability in an Online Store (and Got Paid $50 for It)
  - Author: Nexus0xFault (0xnexus0xfault0x)
  - URL: https://medium.com/@0xnexus0xfault0x/how-a-sneaker-order-led-me-to-an-idor-vulnerability-and-50-d3553a9f9136
  - Date: 2026-10-28
  - Disclosed report: none
  - Raw file: `sources/idor/17-sneaker-order-idor-50.md`

Tags: uuid, unauth, pii-disclosure, api
