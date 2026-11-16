---
title: "IDOR: Account Deletion via Session Misbinding - Attacker Can Delete Victim Account"
author: z3phyrus
url: https://hackerone.com/reports/3154983
date_published: 2025-05-20
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/3154983
bounty: not stated (awarded, amount hidden)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR: Account Deletion via Session Misbinding - Attacker Can Delete Victim Account

Reported to: Mozilla | Reporter: z3phyrus | Reported: 2025-05-20 | Disclosed: 2025-06-03 | State: Resolved | Severity: high

## Report body

## Summary:
A critical vulnerability in the Firefox Accounts API allows an authenticated attacker to permanently delete any user's account by sending a `POST /v1/account/destroy` request using attacker session, but including the victim’s `email` and `authPW` (password hash) in the JSON payload. The server fails to verify that the session making the request belongs to the account being deleted.

## Steps To Reproduce:
1. Login to the victim's account.
[attachment F4367852 not captured]

2. Use Burp Suite to intercept a request to the endpoint https://api.accounts.firefox.com/v1/account/destroy when deleting the account. Capture the JSON body, for example:
```
{
"email": "victims344@gmail.com",
"authPW": "42b4c2940fe2efecce851a2d8e9754d0f1cb1d37e3ccaabb060f9ac21900caff"
}
```
███████

3. Then cancel the request (don't let it reach the server).

4. Login to the attacker's account.
████████

5. Again, use Burp Suite to intercept a request to the same endpoint https://api.accounts.firefox.com/v1/account/destroy when deleting the account. Send it to the Repeater and cancel the request.
██████

6. In the attacker's request, replace the JSON body with the victim's harvested data:
```
{
"email": "victims344@gmail.com",
"authPW": "42b4c2940fe2efecce851a2d8e9754d0f1cb1d37e3ccaabb060f9ac21900caff"
}
```
Send the request.
[attachment F4367860 not captured]

7. The server accepts the request and deletes the victim's account, even if it was from the attacker's session.
[attachment F4367861 not captured]

## Impact

**==Allows attackers to delete victim accounts without permission.==**

Images attached (not captured): F4367852, F4367858 1.png, F4367860 5.png, F4367861 6.png. Some screenshots and the PoC video redacted on the public page.

## Impact (as written)

Allows attackers to delete victim accounts without permission.

## Summary shown at top of the report ("Summary by Mozilla")

An IDOR vulnerability was identified in the Firefox Accounts API endpoint `https://api.accounts.firefox.com/v1/account/destroy` that allows an authenticated attacker using SSO (i.e Google login) to delete a user's account using their email address if they are also registered using SSO. The server fails to verify that the session making the deletion request belongs to the account being deleted.

## Timeline

- 2025-05-20 15:52 UTC: z3phyrus submitted the report to Mozilla.
- 2025-05-21: h1_analyst_mayuresh (HackerOne triage): passed preliminary analyst review.
- 2025-05-23 04:46: h1_analyst_lakshya (HackerOne triage) closed as Informative: "To delete victim's account you need a valid 'authPW' value which is itself a password hash. If attacker has the email and password of the victim, there is no point for this attack."
- 2025-05-23 11:24: frida-k (Mozilla staff) reopened "to take a closer look", status Pending program review; "This issue sounds like a regression of a previous bug", escalated to engineering. Severity updated from critical (9.0) to low: "The issue requires access to the user's credentials which drastically reduces the likelihood of exploitation."
- Reporter follow-up (comment, updated 2025-06-03): new finding. If the victim logs in with SSO and never set a password, an attacker who also has an SSO-only account (no password set) can intercept their own `POST /v1/account/destroy`, change the `email` parameter to the victim's email, and delete the victim's account without knowing credentials. PoC steps: (1) attacker SSO account without password, (2) victim SSO account without password, (3) attacker logs in via SSO, (4) intercept the account-delete request in Burp, send to Repeater, cancel, (5) change `email` to the victim's, (6) send. Conclusion: "Attacker can delete fellow users who login using SSO without resetting password just through user email without access to victim credentials."
- 2025-05-23, frida-k: argues attacker would need access to victim's Gmail to generate authPW. z3phyrus replies: attacker only needs the victim's Gmail address, not access. frida-k: "Ok I can see what you mean now, yes I agree that this would increase the severity of the issue."
- 2025-05-24: frida-k changed status to Triaged, escalated to the responsible team.
- 2025-05-24: z3phyrus requests severity update with CVSS:3.0/AV:N/AC:L/PR:L/UI:N/S:C/C:N/I:H/A:L (9.3 claimed): remote API call, low complexity, only a normal SSO account needed, no user interaction, scope change because SSO is shared across systems, victim account permanently deleted.
- 2025-05-27: ahanphang (Mozilla) updated severity from low to high: "the reporter identified that accounts created with google can be deleted via knowing the victim's email address only. Based on Mozilla's policy, IDORs that bypass authentication or authorization for significant actions, we are setting this finding as a high since it is not possible to take over the account, but to acknowledge that the attacker can delete it."
- 2025-05-27: Mozilla rewarded a bounty (amount hidden). z3phyrus: "This is my first time getting a bounty this big."
- 2025-05-28: frida-k closed as Resolved. 2025-06-03: disclosed and locked.

## Metadata (from report sidebar)

Reported to: Mozilla (managed). Severity: High (7 ~ 8.9). Weakness: IDOR. State: Resolved. Bounty: amount hidden on page. CVE: none.
