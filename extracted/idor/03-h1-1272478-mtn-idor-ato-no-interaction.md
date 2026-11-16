# MTN business portal: profile update with victim ID and username field gives takeover

- **Vuln type / sub-variant:** Numeric/sequential ID IDOR in a profile-update request, with a sensitive identity field (username that equals the email) writable, giving account takeover.
- **Target type:** Authenticated profile-update API on a telecom business portal subdomain.
- **Recon signal:** The profile-update request exposes the user's numeric id in the path. The hunter created two accounts and compared ids, which were adjacent (572 and 573), showing sequential assignment.
- **Root cause:** The update handler used the id from the URL path to select the record to modify and did not check it belonged to the logged-in user. It also allowed the `username` field, which doubles as the login email, to be changed through a generic profile update.
- **Exploitation steps:**
  1. Register two accounts and verify both emails.
  2. As the attacker, intercept a profile update (for example an address change) and send it to a repeater.
  3. Obtain the victim's id (by observing the same request on the victim account in test; in a real attack, by sequential guessing).
  4. Swap the id in the path to the victim's and change the `username` value (leaving the separate `email` field alone) to an attacker-controlled address.
  5. Run password reset to that address and log in as the victim.
- **Bypasses used:** none. The trick is that the identity lives in `username`, not `email`.
- **Impact:** Full account takeover with no user interaction. Reporter argued that a loop over ~500 ids would take over every account on the portal.
- **Bounty:** None.
- **Lesson / pattern:** Check which field actually controls login or reset delivery, not just the field named "email". Sequential ids turn one IDOR into mass takeover.
- **Source:**
  - Title: IDOR Leads To Account Takeover Without User Interaction
  - Author: theranger
  - URL: https://hackerone.com/reports/1272478
  - Date: reported 2021-07-21, disclosed 2022-09-04
  - Disclosed report: https://hackerone.com/reports/1272478
  - Raw file: `sources/idor/03-h1-1272478-mtn-idor-ato-no-interaction.md`

Tags: numeric-id, account-takeover, change-email, forgot-password, enumeration, api
