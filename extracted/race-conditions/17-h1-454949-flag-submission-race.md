# Repeated flag submission inflates Hacker101 CTF points

- **Vuln type / sub-variant:** Limit overrun on a one-time reward claim (points inflation), single-endpoint race.
- **Target type:** Hacker101 CTF platform, flag submission POST.
- **Recon signal:** Points and private-program invitations are granted per solved flag, so a once-per-user-per-flag claim is a natural race candidate.
- **Root cause:** The CTF app did not enforce uniqueness atomically on (user, flag) when recording a claim, so concurrent submissions each counted. Notably, the main HackerOne platform did have a uniqueness check on user and flag ids, so the effect did not propagate there.
- **Exploitation steps:**
  1. Log in and solve one challenge, using the trivial 1-point flag.
  2. Submit the flag normally and capture the POST in Burp.
  3. Replay the POST many times in a tiny window using a multi-threaded script or the Race The Web tool (faster than Burp Intruder).
  4. Observe the same flag credited many times (70 submissions, enough to look like 2 invitations earned).
- **Bypasses used:** None. Triage pushed back on impact, so the author argued the reward-program angle (invitations for 26 points each).
- **Impact:** Low. Inflated CTF scores and completion counters only. Staff found the platform-side uniqueness check blocked real invitations. The first fix removed the score effect but a retest showed the completion column still counted duplicates (13 submissions showed as 13/4).
- **Bounty:** Awarded (amount hidden) in Jan 2019, even though no direct exploit was found, because it triggered a wider race-condition review of the CTF app. Severity Low.
- **Lesson / pattern:** Race bugs in reward and scoring systems get accepted and paid, but impact must be argued in terms of the incentive being abused. Check downstream systems: a duplicated write in one service may be neutralised by a uniqueness check in another. Fixes can be partial, so retest all surfaces that display the counter.
- **Source:**
  - Title: Race Condition in Flag Submission
  - Author: dropper
  - URL: https://hackerone.com/reports/454949
  - Date: 2018-12-04 (disclosed 2019-07-22)
  - Disclosed report: https://hackerone.com/reports/454949
  - Raw file: `sources/race-conditions/17-h1-454949-flag-submission-race.md`

Tags: limit-overrun, single-endpoint, points-inflation, state-corruption, non-atomic, missing-lock
