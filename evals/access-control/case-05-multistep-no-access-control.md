# case-05: Multi-step process with no access control on one step (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step
- **Skill path:** SKILL.md §3 step 4 "force-browse the middle of a multi-step flow out of order"; references/testing-methodology.md §4. Maps to the multi-step sub-variant and the registration-privesc notes (11), and the general "a check on one step is not a check on the next" idea.
- **Date:** 2026-11-19, agent-run live (user-authorized PortSwigger exception).

## What the skill told us to do
When a privileged action is split across steps, the access check often sits on an early step (rendering the form) but not on the final commit. Submit the final step directly as a low-privilege user.

## Steps
1. Logged in as administrator:admin to learn the flow. `POST /admin-roles` with `action=upgrade&username=carlos` returns a confirmation page whose form posts back `action=upgrade&confirmed=true&username=carlos`.
2. Logged out, logged in as wiener:peter.
3. Replayed the **final** step directly: `POST /admin-roles` with `action=upgrade&confirmed=true&username=wiener` -> **200**. The confirmed-commit step has no role check.

## Result
**Green banner: "LAB Solved"** (`.widgetcontainer-lab-status.is-solved`). wiener self-upgraded to administrator.

## Takeaway
Exact match to testing-methodology.md §4: authorization was enforced on the flow's first step, not the state-changing final step. Jumping straight to the commit step bypassed it. No skill gap.
