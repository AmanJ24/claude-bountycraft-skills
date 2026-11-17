# Free plan user creates "read" invite links by tampering access_level (Lovable)

- **Vuln type / sub-variant:** Plan-gated feature enforced only in the UI (subscription tier bypass via request parameter)
- **Target type:** Project sharing / invitation link API (SaaS app builder)
- **Recon signal:** The share dialog told free users that view-only access needs a Pro plan, while still letting them toggle invite links. The gating appeared to be a front-end rule.
- **Root cause:** The endpoint that creates invite codes accepted any access_level value from the client without checking the caller's plan, so the paid-only role could be set by free users.
- **Exploitation steps:**
  1. As a free-plan owner, open a project's share dialog and note that view access is locked.
  2. Disable the invite link, then re-enable it with the proxy intercepting.
  3. On the POST that creates the magic code for the project, change access_level from write to read.
  4. Refresh. The share dialog now shows view access on a free plan.
  5. Accept the invite from a second account and confirm it gets viewer permission.
- **Bypasses used:** Simple body parameter swap on the create-invite request.
- **Impact:** Use of a Pro-only role without paying. The vendor considered it a free use of a paid feature with no security impact and rated it Medium after the researcher cited similar accepted reports. The report was first closed as a duplicate of an earlier one, reopened, then resolved.
- **Bounty:** Not stated (VDP).
- **Lesson / pattern:** Whenever the UI shows a "requires upgrade" lock, replay the underlying call with the locked value. Expect program disagreement on severity for pure entitlement bypass, so frame business impact and cite precedent.
- **Source:**
  - Title: Business Logic Bypass Allows Setting "Read Access" Role Without Pro Plan Subscription
  - Author: ziadmomen
  - URL: https://hackerone.com/reports/3591764
  - Date: 2026-03-08
  - Disclosed report: https://hackerone.com/reports/3591764
  - Raw file: `sources/business-logic/14-h1-3591764-lovable-access-level-bypass.md`

Tags: premium-bypass, subscription-bypass, parameter-tampering, ui-vs-api, client-trust, missing-server-validation, privilege-logic
