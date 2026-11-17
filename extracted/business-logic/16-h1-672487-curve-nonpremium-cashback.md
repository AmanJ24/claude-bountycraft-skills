# Blanking the merchants response lets non-premium users re-pick cashback retailers (Curve)

- **Vuln type / sub-variant:** Response manipulation to reset a one-time, tier-limited selection (premium limit bypass)
- **Target type:** Mobile app rewards feature ("Earn curve cash") and its merchants API
- **Recon signal:** Non-premium users could pick only 3 retailers and then had no way to edit them, while premium users get more. The app seemed to decide editability from the merchants list the server returned.
- **Root cause:** The client trusted the returned list of already selected merchants to decide whether a user may choose again, and the server did not enforce the per-tier limit or the no-edit rule on the update.
- **Exploitation steps:**
  1. As a non-premium user, pick 3 retailers in the rewards feature and confirm.
  2. Reopen the feature with the proxy intercepting the GET that lists the user's program merchants.
  3. Replace the response body with an empty merchants array.
  4. The app now offers selection again. Choose three different retailers and confirm.
  5. Repeat to rotate through all retailers.
- **Bypasses used:** Intercept-and-edit of a server response in a proxy against the Android app.
- **Impact:** Cashback usable across all merchants despite the 3-retailer limit. Triage initially could not reproduce because the "See Retailers" flow redirected to a website. Severity reduced from high to Medium (4.3). The researcher's remediation advice was server-side validation of authorization per endpoint.
- **Bounty:** Awarded plus an extra bonus, amounts hidden.
- **Lesson / pattern:** For mobile apps, treat "you already used your one-time choice" as client state. Blank or alter the status response and see whether the write endpoint independently enforces the rule. Provide a video PoC when triage reproduction differs.
- **Source:**
  - Title: Business Logic Flaw - A non premium user can change/update retailers to get cashback on all the retailers associated with Curve
  - Author: praseudo7
  - URL: https://hackerone.com/reports/672487
  - Date: 2019-08-13
  - Disclosed report: https://hackerone.com/reports/672487
  - Raw file: `sources/business-logic/16-h1-672487-curve-nonpremium-cashback.md`

Tags: response-manipulation, premium-bypass, client-trust, ui-vs-api, missing-server-validation, workflow-bypass
