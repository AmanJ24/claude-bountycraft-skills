---
title: "The IDOR That Got Me My First $3K And Let Me Delete Files Across Tenants"
author: Sid_x95
url: https://medium.com/@Sid_x95/the-idor-that-got-me-my-first-3k-and-let-me-delete-files-across-tenants-be5a65021832
date_published: 2025-11-09
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: $3,000
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# The IDOR That Got Me My First $3K And Let Me Delete Files Across Tenants

## TL;DR

While hunting in a multi-tenant app I found an IDOR that let a _Tenant Admin_ delete files belonging to other tenants by supplying the file's UUID. Regular invited users couldn't do it, but the admin role could. Reported, patched, and I got **$3,000**.

### The setup

- The app is **multi-tenant**: each tenant has folders, files, invited users, and tenant admins.
- Any role could _create_ files; file ownership defaults to the creator's user id.
- Folder access is controlled by tenant admin(s). Files are identified by a **UUID / GUID** (not sequential or guessable).

### What I tried, and what surprised me

First I validated the expected protections:

- An invited user from Tenant A **couldn't** access or delete a file placed in a Tenant A admin-only folder. Good, folder access checks were working.
- I also tried using an invited user from Tenant A to delete a file that belonged to Tenant B, no go. The system rejected that, as expected.

Then I tried the same action with a different hat on:

- Using a **tenant admin** account, I attempted to delete files by their UUID. To my surprise, the admin account could delete files that belonged to other tenants.
- That mismatch, correct protections for basic users but a broken check for admins, was the IDOR.

### Why this mattered

- **High privilege.** Tenant admins have broad capabilities; when their actions aren't properly constrained by tenant boundaries, the impact magnifies.
- **Silent, destructive access.** Deleting files is destructive and may bypass audit trails if not properly logged/validated.
- **Hard to detect by simple scanning.** Because UUIDs aren't iterable, this isn't something you stumble on by brute force, it's an authorization logic flaw that only shows under the right role and flow.
- Because of the role and the destructive nature, the vendor treated it seriously, and the bounty reflected that: **$3,000**.

### Root cause

A graphql endpoint allowed tenant admins to act on files identified by client-supplied UUIDs **without enforcing strict tenant ownership authorization**, an inconsistent authorization check across roles.

### Takeaway

Authorization isn't a checkbox you tick on one endpoint, it's a system-wide contract. I hit a dead end with invited users, kept poking with other roles, found an admin path that skipped the contract, and turned that into a $3,000 lesson: **check every role, every path**.
