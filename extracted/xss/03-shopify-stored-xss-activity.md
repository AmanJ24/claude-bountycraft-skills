# Shopify: stored XSS via staff member name in the activity feed, admin cookie theft

Source completeness: partial (very short report body; exact payload-to-sink wiring and PoC screenshot are the only concrete details).

- **Vuln type / sub-variant:** Stored XSS. HTML-context injection via a user-controlled display name that is rendered unescaped in an admin-facing activity feed. Cross-privilege (low-priv staff to admin).
- **Target type:** Store admin activity/audit log (Shopify admin panel).
- **Recon signal:** A staff member can set an arbitrary member name. The name is later echoed into the activity feed shown to the store admin, a classic second-order sink where attacker-controlled profile text is rendered in a higher-privileged UI.
- **Root cause:** The member name is stored and later written into the activity-feed HTML without output encoding. Unescaped sink: user-supplied name to HTML body context. The injected name breaks out of an attribute (`">`) and adds an element with an event handler. Failed defense: output encoding on the activity feed.
- **Exploitation steps:**
  1. Create a store and add a member (attacker controls a low-privilege staff account).
  2. Set the member name to a payload that closes an attribute and injects an `<svg onload=...>` element.
  3. Perform a store change so an activity entry is generated referencing that member.
  4. When the admin views the activity feed, the stored payload executes in the admin's session.
- **Bypasses used:** None documented (direct unescaped reflection of the stored name).
- **Impact:** Arbitrary JS in the store admin's session; stated goal was theft of admin account cookies, enabling admin account takeover by a staff member.
- **Bounty:** Awarded, amount hidden. Severity High.
- **Lesson / pattern:** Any user-controlled display name that surfaces in an admin activity/audit log is a stored-XSS sink that crosses a privilege boundary; encode names everywhere they are rendered, including log and activity views.
- **Source:**
  - Title: Stored XSS on activity
  - Author: shazadsadiq
  - URL: https://hackerone.com/reports/391390
  - Date: 2018-08-14 (disclosed)
  - Disclosed report: https://hackerone.com/reports/391390
  - Raw file: `sources/xss/03-shopify-stored-xss-activity.md`

Tags: stored, admin-fire, cookie-theft, account-takeover
