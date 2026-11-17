# Flipping subscription-status response booleans unlocks Streamlabs Prime features and free rewards

- **Vuln type / sub-variant:** Client-side trust in a response-derived entitlement flag (response manipulation to bypass premium checks)
- **Target type:** Web dashboard backed by a subscription status API (Logitech program, Streamlabs Prime)
- **Recon signal:** A single endpoint returned a JSON object of subscription flags (all false for a free account). The UI appeared to gate features on those flags, so editing the response was worth a try.
- **Root cause:** The front end decided what to show and allow from the subscription-status response, and some backend endpoints (reward redemption, multistream settings) did not independently verify Prime entitlement. Redemption worked once the UI sent the request.
- **Exploitation steps:**
  1. Observe the subscription status response with every flag false.
  2. Add proxy match-and-replace rules on response bodies: false -> true, and a "blocked":true -> false rule.
  3. Load the rewards dashboard. The redeem button now appears, which it does not for free users.
  4. Redeem, supply an email, and receive a real reward coupon (a gaming mouse).
  5. Also load multistream settings and obtain the RTMP URL and stream key.
- **Bypasses used:** Blanket response rewriting in the proxy. Triage first treated it as a duplicate of an earlier report that only showed a cosmetic feature (custom bot name), until the researcher showed a different, financial impact.
- **Impact:** Free access to Prime features plus unlimited physical-goods coupons, one per account (researcher estimated about $30 per item, scalable by creating accounts). Fixed so that redemption returned 403. Multistream data still reachable at the time of the last comment.
- **Bounty:** Awarded, amount hidden. Severity set to Medium (4.3). Researcher disputed the amount, program cited CVSS scoring.
- **Lesson / pattern:** Response tampering is only valuable if the server accepts the follow-up action. Always click through to the privileged action and quantify the business cost, since a duplicate with weak impact can be reopened by showing a higher-impact consequence.
- **Source:**
  - Title: Manipulating response leads to free access to Streamlabs Prime
  - Author: sudi
  - URL: https://hackerone.com/reports/1070510
  - Date: 2021-01-03
  - Disclosed report: https://hackerone.com/reports/1070510
  - Raw file: `sources/business-logic/13-h1-1070510-logitech-streamlabs-prime-response.md`

Tags: response-manipulation, client-trust, premium-bypass, subscription-bypass, free-services, ui-vs-api, missing-server-validation, financial-loss
