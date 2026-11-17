# Duplicate team invitations via concurrent POST to Omise membership endpoint

- **Vuln type / sub-variant:** Limit overrun on a "once per email" action (duplicate-record creation), single-endpoint race.
- **Target type:** SaaS payment dashboard (Omise), team management form (`POST /team/memberships`), plus a similar link-creation endpoint (`/test/links`).
- **Recon signal:** The app refuses a second invite to an email that is already invited when sent sequentially, which implies a "does this invite exist" check that could be raced.
- **Root cause:** Uniqueness check (is this email already invited?) and the insert of the invitation are not atomic. Parallel requests all pass the check before any row is committed. No unique constraint backs it up.
- **Exploitation steps:**
  1. Log in and send one normal invite to confirm the baseline (a repeat invite is rejected with an error).
  2. Capture the invite POST in Burp, set the protocol to HTTP/1.1 and add a placeholder header so Turbo Intruder accepts it.
  3. Run a Turbo Intruder script with 30 connections, queue 30 copies of the request against a gate, then open the gate so the final bytes go out together.
  4. Observe several `200 OK` responses, then check the pending-members list: the same email appears many times and the invitee gets multiple emails.
  5. Note the duplicates persist even after the invite is accepted. The author also reproduced the same flaw on `/test/links` (same payment link created repeatedly).
- **Bypasses used:** None needed beyond synchronisation (gated last-byte release, 30 concurrent connections, no pipelining).
- **Impact:** Low. Duplicate invitations, email spam to the invitee, polluted team state. Triaged as a business-logic flaw with low severity.
- **Bounty:** Awarded by Omise but amount hidden. Paid before the fix shipped; the fix was released months later (resolved Nov 2021).
- **Lesson / pattern:** Any "this must be unique" create action that errors on sequential repeats is a candidate. Even low-impact duplicate-creation bugs get paid, and the same flaw tends to exist on sibling create endpoints.
- **Source:**
  - Title: Race condition on action: Invite members to a team
  - Author: sim4n6
  - URL: https://hackerone.com/reports/1285538
  - Date: 2021-07-31 (disclosed 2022-03-22)
  - Disclosed report: https://hackerone.com/reports/1285538
  - Raw file: `sources/race-conditions/15-h1-1285538-invite-team-race.md`

Tags: limit-overrun, single-endpoint, turbo-intruder, last-byte-sync, check-then-act, non-atomic, state-corruption
