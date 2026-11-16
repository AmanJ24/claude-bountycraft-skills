---
title: "OTP IDOR: When One Code Verifies the Wrong Phone Number"
author: Parth Narula (scriptjacker)
url: https://scriptjacker.medium.com/otp-idor-when-one-code-verifies-the-wrong-phone-number-df0d2b6475f8
date_published: 2026-09-13
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# OTP IDOR: When One Code Verifies the Wrong Phone Number

Hey Hackers, I am Parth Narula. A penetration tester, bug hunter, red teamer and overall a security researcher. I live for those moments where a bit of out of the box thinking cracks open a critical vulnerability.

This is the story of an IDOR I found in a phone verification flow. The application had two layers of supposed protection. A frontend that restricted which country codes you could pick. And a backend that issued one time passwords before verifying your number. I broke both. Not by finding a single flaw, but by chaining two weaknesses together. With a parameter swap and an OTP meant for a completely different resource, I verified a phone number from a blocked country and walked straight through the onboarding gate.

It started when I was examining the onboarding flow on `REDACTED`. The platform required phone verification before users could access the dashboard. The user interface presented a clean phone number input with a country code dropdown. I noticed the dropdown only offered certain regions. India was not on the list. The platform was explicitly telling users, through its interface, that Indian numbers were not supported.

I selected an allowed country code from the dropdown. Argentina. The number +542345678934. The interface accepted it. I clicked send code and the application fired a POST request to the backend.

I had Burp Suite running. I intercepted the request.

```
POST /api/customer/phone_numbers HTTP/2
Host: REDACTED
Authorization: Bearer my_token
Content-Type: application/json

{
  "number": "+542345678934"
}
```

The server responded with `201 Created`. It generated a verification resource, assigned it a UUID.

I could not enter the OTP as I don't have Argentina phone number.

> But then I asked myself the question I always ask, that What if the backend trusts the frontend too much?

I went back to the add phone number step. I filled in the form again with the allowed Argentina country code. But this time, before clicking send code, I turned on Burp intercept. I caught the outgoing request.

```
POST /api/customer/phone_numbers HTTP/2
Host: REDACTED
Authorization: Bearer my_token
Content-Type: application/json

{
  "number": "+917XXXXXXX2"
}
```

I looked at the JSON body. It contained a single field. Number. The frontend had formatted it with the country code I selected from the dropdown. But the backend was receiving this as plain text inside a JSON object.

I changed the value. I replaced the allowed `+542345678934` with `+917XXXXXXXX2`. An Indian number. A country code that was nowhere to be found in the dropdown. A region the platform explicitly blocked. I kept everything else identical. My authorization token, my headers, the same POST method.

I clicked forward in Burp Suite. The server responded with 201 Created. No error. No rejection. The backend accepted the blocked country code without hesitation. It created a second verification resource, assigned it a new UUID, sent the OTP to the Indian number, and treated the request as completely legitimate.

The OTP arrived on my Indian number. Three copies of it, actually, because I hit resend a couple of times. The code was 249513.

> The first flaw was confirmed. Client side validation was the only thing standing between me and a blocked region. The backend had no idea the frontend was supposed to restrict anything.

Now I had two resources. Resource A, the Argentina number. Resource B, the Indian number. Each with its own UUID. The platform expected me to verify resource B using the OTP sent to +91.

> Then I asked myself the second question I always ask that What if I swap the resource ID?

I took the verification request for resource B. The endpoint that was supposed to confirm the Indian number. I replaced the UUID in the URL from resource B to resource A. The Argentina number. The one that had received a completely different OTP, if it had received one at all.

I kept the verification code identical. 249513. The OTP that was issued for the Indian number.

I clicked send. The server responded with `HTTP/2 200 OK`.

This was the smoking gun. The backend accepted an OTP that was issued for resource B and applied it to resource A. It never checked whether the verification code belonged to the UUID in the URL. It simply saw a valid OTP for the authenticated user and marked the resource as verified.

To confirm this was not just a misleading response, I checked the onboarding dashboard. The platform had moved me to the next step. Add your personal address. The phone verification gate was satisfied. But not by the Indian number. By the Argentina number, verified using an OTP that was never meant for it.

This was a clear and critical issue. The application enforced its restrictions only in the user interface. The API, which is where real security must live, had no idea those rules existed. Worse, the verification logic treated OTPs as global tokens for the user account rather than resource specific secrets. Any OTP could verify any phone number resource you owned.

After I reported this finding, `REDACTED` did not just patch the endpoint. They removed phone verification entirely from their onboarding flow. They rebuilt their authentication system around email and password with magic login links. The phone number step that I had bypassed no longer exists.

When a company tears out an entire authentication mechanism rather than trying to fix it, you know the underlying flaws were fundamental.

The server treated client side validation as sufficient protection for business rules. It assumed that because the dropdown did not show certain country codes, users could not submit them. This is one of the oldest mistakes in application security. Additionally, the verification endpoint failed to implement proper authorization checks. It did not validate that the submitted OTP was issued for the specific resource ID in the URL. An OTP should be cryptographically bound to a single verification resource, not treated as a global account token.

## Lesson Learned

- Never trust client-side restrictions. Always test the backend directly.
- If a value is hidden or blocked in the UI, try modifying it manually in the request.
- For OTP flows, always check what the OTP is actually bound to: user, phone number, resource ID, session, or action.
- Create multiple resources and swap IDs, UUIDs, OTPs, and tokens between them.
- Test relationships between parameters, not just whether each parameter is individually valid.
