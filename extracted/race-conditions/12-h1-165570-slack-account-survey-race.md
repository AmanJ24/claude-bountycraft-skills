# Slack: account-creation survey credit granted multiple times

- **Vuln type / sub-variant:** Limit-overrun (one-time reward claimed repeatedly)
- **Target type:** Post-signup onboarding survey that grants account credit
- **Recon signal:** A one-time survey completion that pays a $100 credit. A reward tied to a "done" flag is a strong overrun candidate. The survey was reachable from the password-reset-complete page, though not always shown.
- **Root cause:** The survey submit handler checks that the survey is not yet completed and then credits the account, non-atomically. Parallel submissions each pass the check before completion is recorded.
- **Exploitation steps:**
  1. Create a new Slack team and reach the survey via the reset-complete URL on the team domain.
  2. Fill the survey and capture the final POST (form-encoded, with a done flag and CSRF crumb).
  3. Replay the request from the shell as parallel curl jobs joined with `&`. First PoC used 2 requests and got $200.
  4. Sending 3 to 4 parallel requests raised the odds that more than one was accepted. Result shown: 2 survey completions, both worth $100.
- **Bypasses used:** None. Shell backgrounding was the delivery mechanism, no special tooling. Notably triagers first could not find the survey in the UI, so the exact URL had to be supplied.
- **Impact:** Free credit multiplied per account, direct financial loss to Slack.
- **Bounty:** Awarded, amount hidden. Fix took about a year after report.
- **Lesson / pattern:** Even crude parallel curl is enough for slow-window races. Send a few more requests than the minimum, and give triagers the precise reachable URL for hidden onboarding flows.
- **Source:**
  - Title: Race Condition in account survey
  - Author: cablej
  - URL: https://hackerone.com/reports/165570
  - Date: 2016-09-03
  - Disclosed report: https://hackerone.com/reports/165570
  - Raw file: `sources/race-conditions/12-h1-165570-slack-account-survey-race.md`

Tags: limit-overrun, check-then-act, non-atomic, financial-loss, free-services, points-inflation
