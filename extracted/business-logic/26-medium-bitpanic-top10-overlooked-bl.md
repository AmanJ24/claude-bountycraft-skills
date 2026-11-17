# Top 10 overlooked business logic flaw classes (checklist article)

- **Vuln type / sub-variant:** methodology/taxonomy (reference article, not a single finding)
- **Target type:** General web applications: e-commerce, auth and reset flows, role-based apps, coupon and checkout systems
- **Recon signal:** Not applicable to one bug. Signals implied: client-supplied price or role fields, promo or quantity limits, multi-step order or approval flows, sequential object ids, and reset tokens that look predictable.
- **Root cause:** The app works technically but fails to enforce the intended business process server-side. Ten classes listed: client-side price tampering; weak password reset logic (incremental tokens, guessable questions); bypassing quantity or redemption limits; flawed role management; skipping multi-step workflow stages; mass assignment of extra fields; IDOR; coupon reuse or expired coupons; session fixation; order-of-operations abuse such as cancelling after receipt but before payment settles.
- **Exploitation steps:**
  1. Intercept checkout and edit price, discount, or quantity fields.
  2. Request later workflow steps directly without completing earlier ones.
  3. Add privileged fields (such as a role) to profile or signup requests.
  4. Replay coupon and redemption calls repeatedly or with expired codes.
  5. Reorder or interrupt transactions (cancel mid-flow) and watch final state.
- **Bypasses used:** Burp parameter edits, direct URL access to final steps, extra JSON fields, request replay, id increment.
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Scanners miss logic flaws because they depend on understanding the app's intended process, so manual testing and threat modeling of each workflow is the way to find them. Suggested defenses: server-side validation and signing of sensitive params, allowlisted fields, server-tracked workflow state, atomic transactions, one-time coupons.
- **Source:**
  - Title: Top 10 Overlooked Business Logic Flaws in Web Applications
  - Author: Spectat0rguy (bitpanic)
  - URL: https://bitpanic.medium.com/top-10-overlooked-business-logic-flaws-in-web-applications-6745d42f3786
  - Date: 2025-01-19
  - Disclosed report: none
  - Raw file: `sources/business-logic/26-medium-bitpanic-top10-overlooked-bl.md`

Tags: methodology, taxonomy, reference, price-manipulation, unlimited-redemption, coupon-stacking, discount-abuse, workflow-bypass, mass-assignment, privilege-logic, order-of-operations, account-takeover, defense
