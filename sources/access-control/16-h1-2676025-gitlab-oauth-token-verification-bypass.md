---
title: "Login email verification bypass via `/oauth/token`."
author: cybxis
url: https://hackerone.com/reports/2676025
date_published: 2024-08-22
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/2676025
bounty: none (Bounty: None)
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #2676025 | Reported to GitLab | Status: Informative (closed; "bug not vulnerability") | Severity: None (0.0), originally high(8.1) | Weakness: Missing Critical Step in Authentication | Bounty: None | Reported: August 22, 2024, 2pm UTC | Disclosed: September 3, 2024. Researcher Medium writeup: https://cybxis.medium.com/a-bypass-on-gitlabs-login-email-verification-via-oauth-ropc-flow-e194242cad96 (scrape separately). Attachments were deleted by GitLab at researcher's request.

## Summary by cybxis

If an app supports ROPC flow, try to bypass any MFA or additional login verification.

Bypass the Gitlab's email verification on login

## Report body (cybxis, August 22, 2024)

### Summary

Hello team, I want to report a security issue on the GitLab authentication functionality. The email verification on login provides an additional layer of security despite 2FA not being implemented. This security measure is being triggered once the account has three or more failed sign-in attempts in 24 hours or a user attempts to sign in from a new IP address. Reference: https://docs.gitlab.com/ee/security/email_verification.html

Gitlab also supports another basic authentication which is the Resource Owner Password Credentials (ROPC) (https://docs.gitlab.com/ee/api/oauth2.html#resource-owner-password-credentials-flow) flow. It is where the user credentials (username and password) will be exchanged for an access token with full read and write scope on the endpoint `/oauth/token`. Just like the web authentication, there is also an implemented restriction on this endpoint to avoid any unauthorized authentication. For example, sending the token exchange request on the said endpoint from a new IP address will result to the following API error response:

```
{"error":"invalid_grant","error_description":"The provided authorization grant is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client."}
```

However, this restriction is only being implemented if there is no active session on the account.

I have discovered that it is possible to bypass this restriction. During my testing, the restriction on `/oauth/token` is being voided once the victim authenticates and completes the email verification regardless of the IP difference between the attacker and the victim.

## Exploit

```
#!/usr/bin/sh

USERNAME=gitlab_username
PASSWORD=gitlab_password

while true
do
curl --data "grant_type=password&username=$USERNAME&password=$PASSWORD&scope=api" \
--request POST "https://gitlab.com/oauth/token"

 echo ''

 sleep 1.5
done
```

The simple bash script above will serve as an event listener for the attacker. Please note that if the account has already an active session (which means it already completed the email verification) the token exchange will be successful on the first request. Otherwise, it will loop with errors until the victim authenticates and completes the verification on any device or IP.

## Current Behavior

- The server only checks whether the account has an active session that has completed an email verification on any device and IP address.

## Expected Behavior

- The server must somehow validate the authenticity of the user sending the request on `/oauth/token`. E.g. sending an email verification link (not code).

## Clarification

This bug only affects accounts with 2FA disabled / no configured.

### Steps to reproduce

For this to reproduce, your GitLab account must trigger the email verification feature on login. To do this, you can do one of the following.

- Logout all your session and restart your pc
- Logout all your session and deliberately commit 3 to 5 incorrect password attempt until you see the warning `Maximum login attempts exceeded. Wait 10 minutes and try again.`. Then wait for 10 minutes to login the correct credentials.

You need two (2) devices with different IP addresses. In my case (my VPS terminal and my local machine).

As an attacker:

1. Login the account via `/oauth/token` endpoint by running the bash script on your terminal to prove the RESTRICTION (IP is flagged and the API will respond an error message mentioned earlier this report).
2. Keep it running and notice that there is no success on the authentication.

As the Victim:
3. Login the account on victim on web login (https://gitlab.com/users/sign_in) which will require an email verification.
4. Complete the verification by providing the verification code sent to the email. Sometimes it is delayed, so you might want to resend the code.

Once the victim is successfully authenticated, the attacker request on the terminal is also being authenticated and able to get access and refresh token.

## Demo and PoC: [redacted/deleted]

## Suggested Fix

The implemented mitigation is somehow not sufficient. Relying on the active sessions simply gives the attacker the success since victims always complete this verification step on their trusted devices. There should be a more strict verification process on the vulnerable endpoint such as sending a verification link on the victim's email address notifying them that an ROPC access request is being attempted from a certain device and IP address.

## Impact

This bug makes the implemented additional layer of security ineffective. This is simply a demonstration where the victim unknowingly completes an email verification for the attacker to access his/her own compromised credentials.

## Comments

### cybxis, Aug 22, 2024 2:54pm UTC

I forgot to change the CVSS -> `S:C` since an attacker can access external sites and other GitLab subdomains that relies on GitLab SSO such as the `customers.gitlab.com` once the account is taken over. The account takeover can be accomplished by adding a new email to the account using the request below.

```
POST /api/v4/user/emails HTTP/2
Host: gitlab.com
Accept: */*
Authorization: Bearer <access_token>
Content-Type: application/json

{"email":"cybxis+attacker2@wearehackerone.com"}
```

The email will then be used later to reset the password.

### cybxis, Aug 23, 2024 7am UTC (updated)

I tried this into 2 different browsers with different IPs. If the victim has an active session and the attacker attempted to authenticate the account on a browser with new IP, the email verification is being triggered and the account is being locked and revokes all active sessions.

However, if the attacker authenticates the account and exchanged for tokens on `/oauth/token` the account is not being locked. The token exchange request will be successful since there is already an active session (that already completed the email verification). The attacker can simply bypass this feature by authenticating the account on `/oauth/token` instead on the web login to bypass this security verification feature.

I also tested this grant flow with OAuth applications and got the same result. This could be due to the fact that it is on the same endpoint.

```
echo 'grant_type=password&username=<your_username>&password=<your_password>'> auth.txt
curl --data "@auth.txt" --user client_id:client_secret \
--request POST "https://gitlab.com/oauth/token"
```

### cybxis, Aug 23, 2024 4:36pm UTC (updated): corrected root cause

Sorry for the wrong analysis about the root cause of this one. This could be more of a problem in regards with the Account email verification. Based on my testing, there wasn't actually any form of verification implemented on `/oauth/token`.

The API does not rely on whether the account has an active session or none but only checks if the account is `active` or `locked`. The ROPC grant flow request will be successful on any IP as long as the account is `NOT LOCKED`.

Steps to prove this:
1. Make sure your account is not locked. Otherwise you have to unlock it by completing the email verification.
2. Run the bash script on your VPS or any terminal with different IP address that you have not yet used from your previous testing (for PoC).
3. You can observe that the server will issue an access and refresh token.
4. Revoke the authorization you have just granted on `https://gitlab.com/-/user_settings/applications`
5. Revoke all sessions on `https://gitlab.com/-/user_settings/active_sessions` and logout the account
6. Run the script again and it will still issue a token. Proving that there is no active session required.

I also observe that the server will continue issuing tokens if the authorization is not revoked despite the account is already locked.

The email verification on login protects GitLab accounts despite 2FA is disabled from any unauthorized access which could lead result to account takeover from a remote attacker / untrusted device. Once a new device/IP is detected, the account will be locked instantly and the user will be notified. But an attacker can bypass the account locking and user notification via `/oauth/token` to takeover the compromised account.

### h1_analyst_shiva (HackerOne triage), Aug 23, 2024: Needs more info

Going through the POC, an attacker would require credentials to victim's account. Also, the Documentation, it does mention that: "The credentials should only be used when: There is a high degree of trust between the resource owner and the client. For example, the client is part of the device operating system or a highly privileged application. Other authorization grant types are not available (such as an authorization code)."

### cybxis, Aug 23, 2024 6:39pm UTC

Yes, that is correct, the point here is that the ROPC can be abused to bypass the Account Email Verification that is being implemented for additional layer of security. The problem I am reporting here is the implemented email verification which is being bypassed by another feature (ROPC). Obviously, there is a conflict between the 2 features.

For the sake of POC, say for example, an account credentials is being leaked with the 2FA disabled. This is still impossible to takeover using other IP or device (other than the devices and IPs being used by the real account owner) via web login due to the Account Email Verification. But by abusing the ROPC endpoint, it is now possible without any further validation and notification.

### cybxis, Aug 24, 2024 5:46pm UTC

I created a new video POC that includes the creation of new VPS to clearly demonstrate this bug. On the video you can see that attempting to login the compromised (assuming) account using new browser and VPN. The account was locked and the email verification is being triggered since the server detected that the login is from a new device/IP. This is just a demonstration of triggering the feature that the attacker wanted to bypass and note that the attacker will not use the web login in the first place due to this verification feature.

## Bypass

Since the attacker knew in the first place that there is an email verification on the web login. The attacker (for POC) created a new VPS and used the ROPC endpoint (instead of the web login) to bypass the triggering of the email verification and able to obtain a fully scoped access token with a refresh token to fully takeover the account without any further verification.

What is being bypassed here is the strict IP and email verification feature.

> The attacker can simply bypass this feature by authenticating the account on `/oauth/token` instead on the web login. This will avoid triggering the email verification which should've protected the account from an unauthorized access.

### h1 triage, Aug 25, 2024: Pending program review (discussing internally with GitLab).

### gmyers (GitLab staff), Sep 3, 2024 5:31pm UTC: closed as Informative

We are opting to close this report as Informative as there is no practical security impact in real-world situations. The ROPC documentation warns that credentials should only be used when there is a high degree of trust and other grant types are not available. Additionally, an attacker must have the victim's password for this "attack" to work, and there are no "steps to reproduce" getting the victim's password. It's reasonable to assume that people won't share active/working account passwords with others, especially when their account is not protected by 2FA - if they do, they can reasonably expect that their account will be compromised as a consequence. I do appreciate your research and tend to agree that any ability to bypass security safeguards is a weakness. However, in this case, the root problem here would be a bug and not a vulnerability. If you would still like to see your concerns addressed, consider opening a public GitLab Issue or Merge Request.

### cybxis, Sep 3, 2024 5:58pm UTC

This is actually confusing on why implement an additional layer of security if it can be bypassed by one of your feature. The email verification on login protects users despite the absence of 2FA implementation on the account. Lets be real, some users don't prefer 2FA and some relies on this email verification as an extra added protection instead of the 2FA. The part "if they do, they can reasonably expect that their account will be compromised as a consequence" is not actually true, as I have demonstrated, it is impossible to takeover or even access the account from new device or IP via web login. But through the ROPC flow, it is possible without any verification from the user.

### cybxis: requested disclosure (Sep 3, 2024). Attachments permanently deleted by GitLab. Severity updated from high(8.1) to none. Disclosed Sep 3, 2024.

### gmyers, Sep 3, 2024 6:06pm UTC

If the behavior you were reporting here was sufficient to authenticate as a user in itself, we'd accept this as a vulnerability. In other words, this would be considered a vulnerability if an attacker could reasonably/reliably obtain their victim's account password. We issue CVEs for self-managed GitLab, and for self-managed GitLab instances running with default settings (e.g. `require_email_verification` + `check_ip_address_for_email_verification` feature flags disabled), having a victim's username and password would be sufficient to authenticate as a user without 2FA enabled - no need for ROPC. This illustrates the true "vulnerability" and root cause of the issue here is the victim taking action to share their password so that the attacker is able to obtain it. The security flaw you've described beyond that would rightly be considered a bug, but not a vulnerability in and of itself.

### cybxis, Sep 3 and Sep 17-18, 2024

Thanks for clarification; accepts that this is a bug rather than a vulnerability. Sep 17: plans to publish a Medium writeup elaborating on the bug, impact and steps without redaction, to make OAuth implementers aware of the ROPC flaw bypassing additional layers such as email verification and MFA (in some cases). Sep 18: posted the writeup link (above).
