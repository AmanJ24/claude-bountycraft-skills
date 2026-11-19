# case-03: URL-based access control can be circumvented (PRACTITIONER) - PASS

- **Lab:** https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented
- **Skill path:** SKILL.md §4 "401/403 request-shape bypass"; references/403-401-bypass.md §2 (URL-override headers `X-Original-URL`/`X-Rewrite-URL`). Directly validates the Clario real-world finding (note 10).
- **Date:** 2026-11-19, agent-run live (user-authorized PortSwigger exception).

## What the skill told us to do
A front-end that blocks a path by URL can be bypassed when the back-end honors `X-Original-URL`/`X-Rewrite-URL`: request an allowed path and smuggle the real one in the header.

## Steps
1. `GET /admin` directly -> **403** (front-end block confirmed).
2. `GET /` with header `X-Original-URL: /admin` -> **200**, renders the admin panel (back-end trusts the header over the real path).
3. Delete carlos: `GET /?username=carlos` with header `X-Original-URL: /admin/delete`. The response line showed 403 but the action processed server-side (admin panel afterward lists only `wiener`, carlos removed).

## Result
**Green banner: "LAB Solved"** (`.widgetcontainer-lab-status.is-solved`), and the admin user list confirmed carlos deleted.

## Takeaway
Exact match to references/403-401-bypass.md §2 and the Clario note (10): the access decision was made on the front-end-visible path while the back-end re-derived it from `X-Original-URL`. The header-bypass arsenal is validated end to end. (Observation worth noting in reporting: the delete returned a 403 status yet still mutated state, so confirm the *effect*, not just the response code, matching SKILL.md §5 false-positive guidance in reverse.)
