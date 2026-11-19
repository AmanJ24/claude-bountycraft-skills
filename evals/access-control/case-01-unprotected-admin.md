# case-01: Unprotected admin functionality (APPRENTICE) - PASS

- **Lab:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality
- **Skill path:** SKILL.md §1 "a hidden, guessable, or role-gated route is reachable directly"; references/recognizing-and-surfaces.md way #3 (security by obscurity); testing-methodology.md §4 (force-browse hidden routes).
- **Date:** 2026-11-19, agent-run live (user-authorized PortSwigger exception).

## What the skill told us to do
Security by obscurity is not access control. Enumerate hidden admin routes (robots.txt, JS, source maps) and force-browse them directly.

## Steps
1. `GET /robots.txt` -> `Disallow: /administrator-panel` (the obscurity tell).
2. `GET /administrator-panel` -> admin page listing users with delete links.
3. Parsed the delete control: `GET /administrator-panel/delete?username=carlos`.
4. Fired it -> 200.

## Result
**Green banner: "LAB Solved"** (`.widgetcontainer-lab-status.is-solved`). Confirmed via DOM status after reload.

## Takeaway
Direct hit on the documented methodology: the admin function had no access control at all, only a non-linked URL. robots.txt did the disclosure. No skill gap.
