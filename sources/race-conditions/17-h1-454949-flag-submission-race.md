---
title: "Race Condition in Flag Submission"
author: dropper
url: https://hackerone.com/reports/454949
date_published: 2018-12-04
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/454949
bounty: not stated (bounty awarded Jan 18, 2019, amount hidden on page)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #454949. Reported to HackerOne (Hacker101 CTF platform) on December 4, 2018, 5:35pm UTC by dropper. Status: Resolved. Severity: Low (0.1 ~ 3.9). Weakness: Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition'). CVE: None. Disclosed July 22, 2019. Duplicates filed: #481352, #489295, #759163.

**Summary:**

This report describes a Race Condition Vulnerability which allow an authenticated user to submit the same Flag multiple times. Increasing the user points and therefore the chances to get an invitation to a private program.

### Steps To Reproduce

To reproduce this bug, you need to:

1. Login with a valid user account
2. Solve one of the challenges and get a Flag. It can be the Trivial one (worth 1 point).
3. Go to the submit page, put the Flag that was found and intercept the POST request that is sent during the Flag submission. You can use Burp Suite for that, for example.
4. Then, you just have to submit the POST several times in a short time frame. I recommend a script with multi threads or "Race The Web" tool that I will mention below.
5. Enjoy your extra points with the same flag.

### Tool Used During PoC

To the PoC I used "Race The Web", available at: https://github.com/insp3ctre/race-the-web
With this tool we can send the requests faster than we could with Burp Intruder. Getting better results.

### Supporting Material/References (Screenshots)

The outcome is shown below: [screenshot F385429: ctfhackerone.png, signed URL removed]

I've used the Trivial Flag 70 times and earned 2 invitations.

### Note

I've created a new user to do this PoC. In case you want to delete, the e-mail is: [REDACTED]

## Impact

An attacker could use this Race Condition vulnerability to earn extra points and increase the chances to get invited to a private program without the need to solve other chalanges available.

## Timeline / comments

**jobert (HackerOne staff)** added weakness "Race Condition" and removed weakness "Violation of Secure Design Principles". (Dec 4, 2018)

jobert changed the status to **Triaged**. (Dec 4, 2018)

Hi @dropper - thanks for your report! This looks like it may be valid. I'll escalate it to the right engineering team to get their feedback on it. We'll get back to you soon. Good luck and happy hacking!

**bencode (HackerOne staff)** (Dec 7, 2018):

Hey @dropper, thanks for finding this. It definitely isn't behaving as expected! After reviewing, we weren't able to find an exploit for this attack. Are you able to see how this could be used maliciously? Or were you able to find any other race conditions that are exploitable?

Thanks!

**dropper** (Dec 7, 2018):

Hi @bencode!

Since the past month (as published in here: https://www.hackerone.com/blog/Hacker101-CTF-Find-flags-get-private-bug-bounty-program-invitations) finding flags at the Hackerone CTF Platform allows the user to directly earn invitations to private bug bounty programs.

"For every 26 points you earn on the CTF, you'll receive another invitation."

So, the issue I am reporting here is not directly related in compromising the web application itself neither the server but in the misuse of the purpose of the reward program.

The issue with this Racing Condition vulnerability is that allows a malicious user to earn those invitations without acctually solving the proposed challenges. As I demonstrated before, I've submitted the Trivial challenge 70 times, getting me 2 invitations. But that could've been many more.

Maybe another way that you can think of this issue is that it is a bypass on the normal flow to earn private invitations :-)

Best regards,

**bencode** (Dec 7, 2018):

@dropper thanks for the reply.

We investigated the integration between the applications and found that even though the CTF shows multiple claims, there is only one reflected on the HackerOne platform due to uniqueness checks on the user_id & flag_id. So there aren't multiple private invitations going out, only the one you legitimately earned.

That being said, there may be other exploits based on the same attack pattern in the CTF application that do have an impact-did you noticed any others?

Thanks,
Ben

**dropper** (Dec 8, 2018):

Hi, @bencode. I understand now and appreciate your clarification. In that case, that would be all for this time. You can close this report. Thanks. Best regards,

**jobert** (Dec 9, 2018):

Hi @dropper - the team is planning to still fix the race condition on ctf.hacker101.com, even though this doesn't result in any new invitations on hackerone.com. We will keep you posted on our progress. Thanks for your patience!

dropper (Dec 9, 2018): Sure, @jobert. No problem! :-)

dropper (Jan 8, 2019): asked for an update, noting this is not high priority.

**HackerOne rewarded dropper with a bounty.** (Jan 18, 2019)

Hi @dropper - We're still waiting for the fix, and will definitely keep you updated.

While there's no immediate exploit for this issue, it did prompt us to do a thorough RCA on race condition of all forms in CTF, and we plan to patch something that would fix the issue across the board. Given that, we've decided to award bounty for your report.

Thanks again, and happy hunting!

dropper (Jan 18, 2019): Thanks for the update and for this bounty.

kotakbasov filed a duplicate (#481352). hkr filed a duplicate (#489295). batee5a joined as participant.

**pei (HackerOne staff)** (April 22, 2019):

Hi @dropper - We have pushed out a fix. Any user who successfully added flags to their account should no longer see those reflected in the website. Could you please verify?

**dropper** (April 23, 2019):

Hi @pei,

To be honest I didn't quite understood what you said about user adding flags and those flags reflected on the website.

But I verified the racing condition issue and it seems to persist on the "completion" column.
I've submitted 13 times the same flag (Micro-CMS v1) and now it shows 13/4.

On the other hand, the points at the progress bar is showing the correct number now: 3/26 points. [screenshot F474110 removed]

**bencode** closed the report and changed the status to **Resolved**. (May 29, 2019) Thanks for your help!

jobert requested to disclose (July 19, 2019). dropper asked that the email in the report be removed before disclosure; jobert redacted it (July 22, 2019). dropper agreed to disclose. Report disclosed July 22, 2019.

abdulsec filed a duplicate (#759163). (Dec 16, 2019)

## Metadata sidebar

Reported on December 4, 2018, 5:35pm UTC. Reported by dropper. Reported to HackerOne (Managed). Severity Low (0.1 ~ 3.9). Weakness: Race Condition (CWE-362). Bounty/Retest/Total reward: Hidden.

Capture status: FULL (images not captured).
