---
title: "Privilege escalation from any user (including external) to gitlab admin when admin impersonates you"
author: skavans
url: https://hackerone.com/reports/493324
date_published: 2019-02-09
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/493324
bounty: not stated (hidden; bounty awarded)
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #493324 | Reported to GitLab | Status: Resolved | Severity: Critical (9 ~ 10) | Weakness: Privilege Escalation | Bounty: Hidden (a bounty was awarded March 11, 2019) | Disclosed: August 26, 2020

## Report (skavans, February 9, 2019, 2:13pm UTC)

**Summary:**
Hey team,
I have discovered a way for any logged in user (attacker) to escalate his privileges to gitlab administrator if the real gitlab administrator impersonates attacker's account.

**Description:**
When the gitlab admin impersonates some user, he gets new `_gitlab_session` cookie and then clicking at `Stop impersonating` he gets his own admin's cookie back. The vulnerability is that the impersonated user (attacker in our case) can see impersonated session at the `Active sessions` so he can switch to it (manually setting it in cookie) and click `Stop impersonating` by himself. This is a way how he can become gitlab administrator.

## Steps To Reproduce:

1. Sign into gitlab app as some user (`attacker`)
2. Go to the active sessions settings tab and revoke all the sessions besides the current active one
3. Sign into gitlab app in other browser as administrator (`admin`)
4. Go to users admin section and impersonate `attacker` user
5. Update the active sessions tab as `attacker` and make sure the second session appeared there (this is the admin logged into your account) [screenshot F420971: Untitled.jpeg]
6. Inspect the `Revoke` button and make sure you see the session ID there. Copy it. [redacted screenshot]
7. Go to index page of gitlab as `attacker` (http://gitlab.bb/ in my case), I do not know why, but it is important step
8. Clear `attacker` browser's cookie
9. Open the developer console as `attacker` and manually set `_gitlab_session` to the copied one with:
   ```
   document.cookie ="_gitlab_session=[REDACTED]";
   ```
9. Refresh the attacker's page and make sure you are now inside the impersonated session [screenshot F420978: Untitled2.jpeg]
10. Click `Stop impersonating` at the top-right corner as `attacker` and make sure you are now logged in as gitlab admin. [redacted screenshot]

## Impact

Every gitlab authenticated user can escalate his privileges to admin ones and give complete access to all gitlab services, projects and abilities. Only he needs to do is ask admin to impersonate his account because of something works bad there.

## Mitigations (skavans comment)

I think you should either not to show the impersonated sessions inside the active sessions tab or use some ID other than gitlab_session itself in revoking process to not let attacker know this session ID.

## Timeline / comments

- Feb 9, 2019: title changed from "...when admin impersonating you" to "...when admin impersonates you".
- Feb 9, 2019 (gitlab-securitybot): automated acknowledgment.
- Feb 10, 2019 (skavans): The severity is Critical (9.0) in fact, right CVSS vector includes User Interaction: Required, Attack Complexity: Low; I mischose this components when submitting.
- Feb 11, 2019 (dappelt, GitLab staff): Triaged. "Good finding! We have verified this finding and have escalated to our engineering team. Tracking internally at https://gitlab.com/gitlab-org/gitlab-ce/issues/57534. Issue public 30 days following release of a patch."
- Feb 22/23, 2019 (securitybot): fix ETA scheduled 2019-04-22 then moved to 2019-03-22.
- Mar 11, 2019: dappelt updated severity from high(8.0) to critical.
- Mar 11, 2019: GitLab rewarded skavans with a bounty (amount hidden): "Your finding has been patched in the recent GitLab security release (https://about.gitlab.com/2019/03/04/security-release-gitlab-11-dot-8-dot-1-released/) and we are awarding a bounty." Report resolved.
- Aug 11/13, 2020: skavans requested disclosure. Aug 26, 2020: dcouture agreed; report disclosed.
