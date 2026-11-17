# Duplicated retest payouts by racing the HackerOne retest confirmation

- **Vuln type / sub-variant:** Limit overrun on a payment-issuing action (double payout), single-endpoint race.
- **Target type:** Bug bounty platform (HackerOne), retest confirmation request triggered from a retest-request email.
- **Recon signal:** A one-shot action that creates a payment record when confirmed. Money-producing "once" actions are the obvious race target.
- **Root cause:** The confirm-retest handler did not serialise or deduplicate the payout creation, so concurrent confirmations each created a payment activity. Staff classified it as improper synchronisation on a shared resource (CWE-362).
- **Exploitation steps:**
  1. Receive a retest request email and open the confirm action with the proxy on.
  2. Copy the confirm request as a curl command.
  3. Run it about 5 times concurrently by chaining background curl invocations with `&`.
  4. Check the bounties settings page: the retest payment shows up repeated under retest payments.
  5. After a couple of weeks, a $500 payment actually landed in the bank account, proving the duplicates were real money rather than display artifacts.
- **Bypasses used:** None. Plain concurrent curl (no single-packet tooling) was enough, which shows the window was wide.
- **Impact:** Direct financial loss to the platform. The extra payments went unnoticed for about two weeks, and staff only noticed because a data migration to remove the redundant activities broke the billing overview.
- **Bounty:** About $2,100 net: $2,500 bounty, minus $500 of duplicated retest payout, plus the $100 retest reward. Severity Medium (4.2).
- **Lesson / pattern:** Payout and credit endpoints deserve race testing even with crude tooling. Proof can be strengthened by showing the money really arrived. Also note that fixes get verified through a retest, and the retest here paid only once.
- **Source:**
  - Title: Race condition in performing retest allows duplicated payments
  - Author: cablej (Jack Cable)
  - URL: https://hackerone.com/reports/429026
  - Date: 2018-10-26 (disclosed 2018-12-27)
  - Disclosed report: https://hackerone.com/reports/429026
  - Raw file: `sources/race-conditions/16-h1-429026-retest-race.md`

Tags: limit-overrun, single-endpoint, double-spend, financial-loss, non-atomic, missing-lock, check-then-act
