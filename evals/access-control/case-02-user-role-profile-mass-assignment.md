# case-02: User role can be modified in user profile (APPRENTICE) - PASS

- **Lab:** https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile
- **Skill path:** SKILL.md §1 "a client-supplied identifier/claim names the privilege" + §4 self-assign role field; references/recognizing-and-surfaces.md way #1/#4; testing-methodology.md §2 (self-targeted role fields). Maps to real notes 08 (read-only->admin via self-PUT) and 21 (Scheduler self-assigns `team_leader_ids`).
- **Date:** 2026-11-19, agent-run live (user-authorized PortSwigger exception).

## Why this lab (swap note)
The planned case-02 "User role controlled by request parameter" is cookie-based (`Admin=true`). In this harness the server set the `Admin` cookie such that a JS `document.cookie` write would not overwrite it (a generic `testcookie` wrote fine, so it is a per-cookie/HttpOnly quirk, not a skill issue, and `fetch` cannot set the forbidden `Cookie` header). This sibling lab exercises the identical skill branch (client-supplied role, self-assign) through a JSON body, which is cleanly driveable, so it was used instead.

## Steps
1. Logged in as wiener:peter.
2. Profile update endpoint: `POST /my-account/change-email` taking a JSON body.
3. Injected the role field the UI never exposes: `{"email":"wiener@test.com","roleid":2}` -> 200, mass-assigned.
4. `GET /admin` now authorized (carlos visible); fired `GET /admin/delete?username=carlos` -> 200.

## Result
**Green banner / solved** confirmed (`is-solved` on reload).

## Takeaway
Textbook self-assigned-privilege (mass assignment of a role field the server trusts from the client). Exactly the pattern the skill says to test on profile/role endpoints. No skill gap.

## Harness note
Cookie-based access-control labs where the target cookie is HttpOnly cannot be driven by in-page JS (`document.cookie` write ignored for that cookie; `fetch` cannot set `Cookie`). Drive those with Burp instead. Non-cookie access-control labs (body/header/URL/method) are fully driveable.
