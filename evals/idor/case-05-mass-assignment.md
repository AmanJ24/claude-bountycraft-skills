# Eval: Exploiting a mass assignment vulnerability (false-positive-adjacent: mass assignment)
- **Target:** https://portswigger.net/web-security/api-testing/lab-exploiting-mass-assignment-vulnerability
- **Sub-variant:** mass assignment (NOT object-ownership IDOR; skill section 5 flags this family)
- **Skill path:** SKILL section 5 names mass-assignment/trusted-client value as a distinct family; attack surface = hidden object fields from API responses. GET /api/checkout to see the order object, POST adding a chosen_discount field to buy the jacket.
- **Pass criteria:** [x] skill identifies hidden field from the GET response; [x] injects it on POST; [x] buys the jacket
- **Mode:** live (agent-run, user-authorized)
- **Result:** PASS (agent-run live, 2026-11-16). GET /api/checkout exposed hidden chosen_discount.percentage field; POST /api/checkout with percentage:100 ordered the jacket. Confirms SKILL section 5: mass assignment is a distinct family, found by inspecting API response fields.
