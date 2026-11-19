# Scheduler to Team Admin: UI says "only editors can create teams," the API does not, so a low-priv token creates and controls teams

- **Vuln type / sub-variant:** vertical privilege escalation via missing function-level access control (MFLAC); the create/copy/assign API endpoints enforce no role check though the UI hides them from low-privileged (Scheduler) users
- **Target type:** SaaS team-management platform (role-based: Editors/Admins manage structure, Schedulers only assign people), REST API at `api.examplecenter.com`
- **Recon signal:** the UI shows "Only editors can create teams" and hides the button; watching Burp traffic on tab switches reveals the team endpoints the UI calls. Create/Copy/Assign operations are the usual hiding spots for privesc.
- **Root cause:** authorization enforced only in the front end; the backend API blindly accepts the request and does not verify the caller's role before creating teams or assigning leadership
- **Exploitation steps:**
  1. As a Scheduler, send `POST /api/services/v2/service_types/{id}/teams` with your Scheduler bearer token and a body creating a team with `team_leader_ids: [your_id]`.
  2. Expected 403, got `200 OK`: team created and you are assigned Team Leader.
  3. Escalate with `copy_from: "<team_id>"` to clone an existing team's full structure, members, and config, assigning yourself leader.
  4. Set `team_leader_ids` to assign yourself or others as leaders, controlling the hierarchy.
  5. No rate limiting: create hundreds of teams; duplicate names allowed (no uniqueness check).
- **Bypasses used:** calling the hidden API directly with a low-privilege token; self-targeted `team_leader_ids`; `copy_from` to inherit structure
- **Impact:** Scheduler gains admin-like control (create/copy teams, assign leadership), plus operational abuse (mass creation, duplicate names). Severity High to Medium; fixed to return 403.
- **Bounty:** $700 + $100 retest bonus ($800 total)
- **Lesson / pattern:** never trust UI restrictions, test the API; backend authorization is the real security layer. Replay every create/copy/assign action from a low-privilege session, and check self-assignment parameters (`*_leader_ids`, role fields) plus missing rate limits that amplify impact.
- **Source:**
  - Title: $800 Bounty: Privilege Escalation via API, From Scheduler to Team Admin
  - Author: Abhi Sharma (@a13h1)
  - URL: https://infosecwriteups.com/800-bounty-privilege-escalation-via-api-from-scheduler-to-team-admin-810bb8401a0f
  - Date: 2026-04-23
  - Disclosed report: none
  - Raw file: `sources/access-control/21-infosecwriteups-api-scheduler-to-admin.md`

Tags: vertical-privesc, mflac, front-end-only, api-authz, self-assign-role, copy-operation, missing-rate-limit, saas-rbac
