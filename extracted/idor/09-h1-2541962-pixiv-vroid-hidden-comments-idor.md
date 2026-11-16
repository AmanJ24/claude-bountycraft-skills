# VRoid Hub: liking a hidden comment by id returns its content

- **Vuln type / sub-variant:** Numeric comment-id IDOR through a side-effect endpoint (like/heart action) whose response echoes the object. Read access via a write-style call.
- **Target type:** Social features API of a 3D character sharing site (comments on media posts).
- **Recon signal:** A like/heart request takes a status (comment) id in the path. The hunter reasoned that hidden comments still exist server-side and might still accept actions by id.
- **Root cause:** The like endpoint only looked up the comment by id and did not check visibility (comments disabled by the owner) or permission to see it. Its response returned the comment body and author, bypassing the owner-only visibility rule.
- **Exploitation steps:**
  1. Victim posts media with comments allowed, a comment is made, then the victim disables comments.
  2. Obtain the comment id (in the test, from page source `entityIds`; reporter says a real attacker would not need this step, presumably by guessing or other exposure).
  3. As the attacker, replay any POST with the path set to `/api/statuses/<id>/hearts` and an empty JSON body.
  4. Read the comment content and owner in the response.
- **Bypasses used:** none. Using the like action instead of the blocked read path.
- **Impact:** Disclosure of any user's hidden comments, text and author. Rated medium (4.6).
- **Bounty:** $500, plus $50 for the retest. Fix returned a not-found error for hidden comments.
- **Lesson / pattern:** Visibility rules are often enforced only on the read/list endpoint. Try every other action endpoint (like, report, reply, share) on the hidden object id and inspect the response body.
- **Source:**
  - Title: Disclose Hidden Comments on Media Section of hub.vroid.com
  - Author: giwadaoud
  - URL: https://hackerone.com/reports/2541962
  - Date: reported 2024-06-07, disclosed 2026-01-18
  - Disclosed report: https://hackerone.com/reports/2541962
  - Raw file: `sources/idor/09-h1-2541962-pixiv-vroid-hidden-comments-idor.md`

Tags: numeric-id, api, pii-disclosure, enumeration
