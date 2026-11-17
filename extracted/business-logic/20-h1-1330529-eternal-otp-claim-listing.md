# OTP and requestId not bound to resource id: claim any unclaimed restaurant listing (Zomato)

- **Vuln type / sub-variant:** OTP bypass via unbound verification token (cross-resource OTP reuse), leading to listing takeover
- **Target type:** Merchant onboarding / "claim your listing" flow (send-auto-claim-otp and verify-auto-claim-otp endpoints)
- **Recon signal:** A two-step claim flow where the send step takes a phone number plus a `resId`, and the verify step takes the code, a `requestId`, and a `resId` again, so the resource id is user-supplied in both steps.
- **Root cause:** The verify endpoint validated the OTP and requestId pair but did not check that they were issued for the same `resId` now being claimed. An OTP legitimately issued for the attacker's own restaurant therefore validated against any other restaurant.
- **Exploitation steps:**
  1. Attacker holds their own restaurant page with a mapped mobile number and email (prerequisite).
  2. Call the send-OTP endpoint with the attacker's own phone number and own `resId`; receive the OTP on their phone and a `requestId` in the response.
  3. Call the verify endpoint with that OTP and `requestId`, but swap `resId` for an unclaimed victim restaurant.
  4. Server maps the attacker's email as owner or manager of the victim listing.
- **Bypasses used:** Parameter swap on the second step only (resource id substitution); reuse of a token across objects. Only works for listings without an existing owner.
- **Impact:** Takeover of unclaimed non-delivery restaurant listings. Program raised severity to Critical (9.1).
- **Bounty:** $3,250 total ($2,000 on triage, $750 bounty, $500 bonus).
- **Lesson / pattern:** In any multi-step token flow, bind the token to every identifier it authorizes (user, resource, action) and re-check on verify. Test by getting a valid token for something you own, then replaying the verify step with someone else's object id.
- **Source:**
  - Title: Claiming the listing of a non-delivery restaurant through OTP manipulation
  - Author: ashoka_rao
  - URL: https://hackerone.com/reports/1330529
  - Date: 2021-09-05
  - Disclosed report: https://hackerone.com/reports/1330529
  - Raw file: `sources/business-logic/20-h1-1330529-eternal-otp-claim-listing.md`

Tags: otp-bypass, parameter-tampering, missing-server-validation, privilege-escalation, account-takeover, assumption-flaw
