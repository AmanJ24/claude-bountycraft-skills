# HackerOne hacker101 CTF: double group join creates an undeletable member

- **Vuln type / sub-variant:** State-corruption via TOCTOU on a join action (duplicate membership rows)
- **Target type:** Group invitation-link join endpoint (ctf.hacker101.com)
- **Recon signal:** The researcher had just been studying race conditions and saw another disclosed race on the same platform, so tried a once-only join action.
- **Root cause:** Time-of-check to time-of-use on membership. Join checks that the user is not already a member, then inserts. Parallel joins both pass the check and create two membership entries, after which removal logic (which likely expects one row) fails.
- **Exploitation steps:**
  1. Use two accounts: a group leader who creates a group and invite link, and a normal member.
  2. As the member, intercept the join POST (carries CSRF token and invite token).
  3. Replay it in parallel, using the Turbo Intruder race example script with concurrency lowered from 30 to 5 or 10. With 5 parallel requests the member got added twice.
  4. Try removing the user as leader or self-leaving: removal always fails.
  5. Side effect: group statistics were also skewed.
- **Bypasses used:** None. Needs a valid invite link, which can be copied from page source and reused by another account.
- **Impact:** Permanent, irremovable group membership. The leader's only recourse is deleting the group. Low severity (2.4) since an invite is required.
- **Bounty:** Awarded 2019-08-16, amount hidden. Fix shipped about 7 months later.
- **Lesson / pattern:** Races can corrupt state, not just overrun limits. A duplicated record can break later operations (delete) and the damage persists. Lower thread counts (5 to 10) can work better than 30 when over-concurrency causes failures.
- **Source:**
  - Title: Race Condition leads to undeletable group member
  - Author: yashrs (Yash Sodha)
  - URL: https://hackerone.com/reports/604534
  - Date: 2019-06-09
  - Disclosed report: https://hackerone.com/reports/604534
  - Raw file: `sources/race-conditions/14-h1-604534-undeletable-group-member-race.md`

Tags: state-corruption, toctou, social-action, turbo-intruder, send-group-parallel, check-then-act, non-atomic
