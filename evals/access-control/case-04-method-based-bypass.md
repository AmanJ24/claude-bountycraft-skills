# case-04: Method-based access control can be circumvented (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented
- **Skill path:** SKILL.md §4 "401/403 request-shape bypass" (method change); references/403-401-bypass.md §1 (HTTP method/verb fuzzing). Maps to the general MFLAC tells (notes 02, 08, 21).
- **Date:** 2026-11-19, agent-run live (user-authorized PortSwigger exception).

## What the skill told us to do
A guard mapped to one verb may not cover others. If `POST` to a privileged endpoint is role-checked, try the same action as `GET` with query params.

## Steps
1. Logged in as administrator:admin, read `/admin`: the role-change control is `POST /admin-roles` with `username` + `action=upgrade`.
2. Logged out, logged in as wiener:peter.
3. `POST /admin-roles` (body `username=wiener&action=upgrade`) as wiener -> **401** (method+role check enforced on POST).
4. Same action as **GET**: `GET /admin-roles?username=wiener&action=upgrade` -> **200** (the check does not cover GET). wiener is now administrator.

## Result
**Green banner: "LAB Solved"** (`.widgetcontainer-lab-status.is-solved`).

## Takeaway
Exact match to references/403-401-bypass.md §1: the authorization check was bound to the HTTP method, not the action. Swapping POST->GET (carrying the body as query params) bypassed it. No skill gap.
