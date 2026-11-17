# 2FA requirement enforced on report submitter but not on invited collaborator (HackerOne)

- **Vuln type / sub-variant:** Security control applied to the primary actor but not to a secondary path to the same data (2FA requirement bypass via collaborator invite)
- **Target type:** Bug bounty platform, program setting "require 2FA to report" combined with report collaboration
- **Recon signal:** A program-level policy demanded 2FA for submitters, while a separate feature let a submitter invite another hacker onto the same report.
- **Root cause:** The 2FA check lived in the report-submission path only. The collaborator-invite acceptance path never re-evaluated the program's 2FA requirement for the invitee.
- **Exploitation steps:**
  1. Create account A with 2FA and account B without 2FA.
  2. Pick a program that mandates 2FA and permits collaboration.
  3. As A, submit a report and add B as collaborator.
  4. B accepts the invite and reads the report with no 2FA.
- **Bypasses used:** Alternate access path (invitation) instead of attacking the check directly.
- **Impact:** Claimed exposure of sensitive vulnerability data to a non-2FA account. Closed as duplicate of an earlier report; triage stated the requirement was designed for submitters, not invitees, so the vendor viewed it as intended scope.
- **Bounty:** None (duplicate).
- **Lesson / pattern:** When a policy gate exists, enumerate every other route to the same resource (invites, shares, API tokens, exports) and check whether each re-applies the gate. Expect pushback when the vendor defines the control's scope narrowly, and frame impact against the stated purpose of the control.
- **Source:**
  - Title: Business Logic error leads to bypass 2FA requirement
  - Author: abdulprkr
  - URL: https://hackerone.com/reports/2571981
  - Date: 2024-06-24
  - Disclosed report: https://hackerone.com/reports/2571981
  - Raw file: `sources/business-logic/21-h1-2571981-hackerone-2fa-requirement-bypass.md`

Tags: 2fa-bypass, workflow-bypass, assumption-flaw, data-exposure, missing-server-validation
