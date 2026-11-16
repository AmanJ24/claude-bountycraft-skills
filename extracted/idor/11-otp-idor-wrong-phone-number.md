# OTP issued for one phone resource accepted for another (verification misbinding)

- **Vuln type / sub-variant:** OTP/verification misbinding IDOR using resource UUIDs in the verify URL, chained with server-side trust of client-side validation (blocked country code).
- **Target type:** Onboarding phone verification API (`POST /api/customer/phone_numbers`, then a verify call keyed by the phone-number resource UUID). Target redacted.
- **Recon signal:** The country-code dropdown excluded certain regions (India), implying a UI-only restriction. Creating a phone resource returned a UUID, which suggested a per-resource verify endpoint where IDs could be swapped. Hunter habitually asks "what is the OTP actually bound to?"
- **Root cause:** Two missing checks. (1) The create endpoint did not validate the `number` field server-side against the allowed regions. (2) The verify endpoint did not check that the submitted code was issued for the phone resource named in the URL; it treated any valid OTP for the authenticated user as valid for any of the user's resources.
- **Exploitation steps:**
  1. Create phone resource A with an allowed-country number via the normal UI flow (cannot receive its SMS).
  2. Create resource B by intercepting the same POST and replacing `number` with a blocked-region number the hunter controls; server returns 201 with a new UUID and sends the OTP there.
  3. Take the verify request for B, change the resource UUID in the URL to A, keep the OTP received for B.
  4. Server returns 200 and marks A verified; onboarding advances to the next step.
- **Bypasses used:** Burp interception to override the dropdown-limited value; swapping resource IDs across two owned objects.
- **Impact:** Bypass of the phone verification gate and region restriction (verify a number you cannot receive codes for). Vendor later removed phone verification entirely and moved to email magic links. This is a business-logic/verification bypass, not cross-user data access.
- **Bounty:** Not stated in source.
- **Lesson / pattern:** Create two resources you own and cross-wire their IDs and tokens; OTPs must be bound to the specific resource, not just the user.
- **Source:**
  - Title: OTP IDOR: When One Code Verifies the Wrong Phone Number
  - Author: Parth Narula (scriptjacker)
  - URL: https://scriptjacker.medium.com/otp-idor-when-one-code-verifies-the-wrong-phone-number-df0d2b6475f8
  - Date: 2026-09-13
  - Disclosed report: none
  - Raw file: `sources/idor/11-otp-idor-wrong-phone-number.md`

Tags: otp, uuid, api, session-misbinding
