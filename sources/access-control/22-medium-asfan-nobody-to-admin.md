---
title: "From Nobody to Admin: A Short, Practical Guide to Privilege Escalation .."
author: Muhammed Asfan (@MuhammedAsfan)
url: https://medium.com/@MuhammedAsfan/from-nobody-to-admin-a-short-practical-guide-to-privilege-escalation-b6749f3ec9a7
date_published: 2025-12-27
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- Scrape note: full article publicly readable (not member-only). Generic checklist-style guide, not a specific disclosed bug. -->

2 min read. Dec 27, 2025. Tags: Bug Bounty Tips, Bugbounty Writeup, Cybersecurity, Privilege Escalation, Broken Access Control.

"Imagine you sign up as a normal user...and two minutes later you're editing the admin's settings without ever touching their password."

That moment is called **privilege escalation** - and it's where many "meh" bugs quietly turn into real money.

## What Is Privilege Escalation?

Privilege escalation is just **climbing the access ladder**:

- **Vertical:** user -> admin, read-only -> read/write, web user -> server root.
- **Horizontal:** same level, but into **someone else's** data (User A sees/edits User B).

Why it matters:

- It turns small bugs (IDOR, weak checks, misconfigs) into:
- Account takeover
- Org-wide compromise
- Full system pwn

## Web / Bug Bounty Priv-Esc: Fast Playbook

Log in as a low-privileged account and think:

"Can I act like an admin or another user if I just change the data I send?"

## 1. Attack the IDs

- Tweak IDs in URLs/body:
- `user_id`, `account_id`, `org_id`, `team_id`, `project_id`, `invoice_id`
- If your `id=100`, try `id=101`, `id=1`, etc.
- If you can read/change others' resources -> access control/privilege escalation.

## 2. Replay Admin Requests

- Capture an admin-only action (role change, user management, billing).
- Swap in your **normal user cookies** and replay the exact same request.
- If it still succeeds -> missing server-side role check.

## 3. Ignore the UI, Hit the API

- Hunt endpoints like `/admin/*`, `/internal/*`, `/manage/*`, `/settings/*`.
- Even if the button is hidden, call the endpoint directly as a normal user.
- Working calls here = vertical privilege escalation.

## 4. Break Tenant Boundaries

- In multi-tenant apps, flip org/workspace IDs:
- `/org/1/users` -> `/org/2/users`
- Seeing or editing another org's data is a high-impact escalation.

## 5. Abuse "Temporary" Elevation

- Features to watch:
- Invites (workspaces, orgs, teams)
- "Act as user" / impersonation tools
- OAuth apps with broad scopes
- Try:
- Reusing invite/impersonation links
- Keeping access after removal/demotion
- Using tokens in places they weren't meant for

## Pocket Checklist (Drop into Your Notes)

- Change resource IDs and org IDs.
- Replay admin actions with low-priv cookies.
- Call hidden `/admin` / `/manage` endpoints directly.
- Abuse invite, impersonation, and OAuth flows.
- Check if old sessions/tokens still have high privileges after demotion.

Privilege escalation isn't about fancy payloads; it's about thinking,

**"What if I simply refuse to stay in my assigned lane, and the app lets me?"**
