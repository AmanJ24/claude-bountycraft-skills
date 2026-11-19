# Privilege-escalation playbook: refuse to stay in your assigned lane and see if the app lets you

- **Vuln type / sub-variant:** vertical and horizontal privilege-escalation methodology (checklist/guide, not one disclosed bug): id tampering, admin-request replay, hidden-endpoint access, tenant-boundary breaks, abuse of temporary elevation
- **Target type:** multi-tenant web apps with roles, admin panels, invites, impersonation, and OAuth apps
- **Recon signal:** after signing up as a low-privileged user, ask "can I act like an admin or another user just by changing the data I send?"; watch for predictable ids (`user_id`, `org_id`, `team_id`, `invoice_id`), hidden `/admin` `/manage` `/internal` routes, and invite or impersonation links
- **Root cause:** missing server-side authorization: role checks only in the UI, object/tenant ids not scoped to the caller, and elevated access that persists after demotion or removal
- **Exploitation steps (fast playbook):**
  1. Attack the ids: increment or swap `id=101`, `id=1`, flip `org_id`; reading or editing others' resources is privilege escalation.
  2. Replay admin requests: capture an admin-only action, resend it with a normal user's cookies; success means no server-side role check.
  3. Ignore the UI, hit the API: call `/admin/*`, `/manage/*`, `/settings/*` directly even when the button is hidden.
  4. Break tenant boundaries: change `/org/1/users` to `/org/2/users` to reach another org's data.
  5. Abuse temporary elevation: reuse invite/impersonation links, keep access after removal/demotion, use tokens where they were not intended.
- **Bypasses used:** direct id/tenant manipulation, cookie-swap request replay, direct API calls past a hidden UI
- **Impact:** account takeover, org-wide compromise, full system compromise depending on the function exposed
- **Bounty:** not stated (methodology guide)
- **Lesson / pattern:** privilege escalation is not about payloads; it is about enforcing authorization on every request server-side. Test each admin-capable action and each object/tenant id from a low-privilege session, and re-check whether old sessions or tokens stay privileged after a demotion.
- **Source:**
  - Title: From Nobody to Admin: A Short, Practical Guide to Privilege Escalation
  - Author: Muhammed Asfan (@MuhammedAsfan)
  - URL: https://medium.com/@MuhammedAsfan/from-nobody-to-admin-a-short-practical-guide-to-privilege-escalation-b6749f3ec9a7
  - Date: 2025-12-27
  - Disclosed report: none
  - Raw file: `sources/access-control/22-medium-asfan-nobody-to-admin.md`

Tags: vertical-privesc, horizontal-privesc, idor, request-replay, hidden-endpoint, multi-tenant, temporary-elevation, methodology
