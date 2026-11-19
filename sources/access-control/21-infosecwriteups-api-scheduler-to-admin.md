---
title: "$800 Bounty: Privilege Escalation via API - From Scheduler to Team Admin"
author: Abhi Sharma (@a13h1)
url: https://infosecwriteups.com/800-bounty-privilege-escalation-via-api-from-scheduler-to-team-admin-810bb8401a0f
date_published: 2026-04-23
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: $700 + $100 retest bonus ($800 total)
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- Scrape note: full article publicly readable. Images omitted. Bearer token in PoC is a placeholder. -->

Published in InfoSec Write-ups. 4 min read. Apr 23, 2026. Tags: Bug Bounty, Infosec, Cybersecurity, Broken Access Control, Privilege Escalation.

Hi Everyone! I recently discovered a **Broken Access Control / Privilege Escalation vulnerability** in a SaaS platform (ExampleCenter) that allowed a low-privileged user (Scheduler role) to perform **editor-level administrative actions** via direct API calls.

For this discovery, I was awarded a **$700 bounty along with a $100 bonus for retesting**. Let's explore how this bug worked, how I exploited it, and why it was important.

## Understanding the Target

ExampleCenter is a SaaS platform used for managing teams, scheduling, and workflows within organizations. It follows a role-based access control model:

- **Editors/Admins** -> Can manage teams, structure, and permissions
- **Schedulers** -> Can only assign people to existing teams

To maintain proper security boundaries, only **Editors and Admin** should be able to:

- Create teams
- Copy teams
- Assign team leaders
- Modify team configurations

The UI correctly enforces this restriction by displaying:

> "Only editors can create teams."

At this point, everything looked correct.

But from experience, I've learned one important thing:

> If something is blocked in the UI... it doesn't mean it's blocked in the backend.

## The Moment of Curiosity

Instead of stopping there, I opened Burp and started looking at how the application was communicating with the backend.

I noticed that when switching tabs or loading team-related data, the app was calling certain API endpoints. That gave me an idea:

What if the team creation functionality still exists in the API, but is just hidden in the UI?

So I decided to test it manually by accessing the team creation api from higher level user account and use via scheduler user.

## The Flaw: Backend Didn't Enforce Authorization

I crafted a request to the team creation endpoint using my **Scheduler account token**:

```
POST /api/services/v2/service_types/{id}/teams HTTP/2
Host: api.examplecenter.com
Authorization: Bearer <scheduler_token>
Content-Type: application/json

{
  "data": {
    "type": "team",
    "attributes": {
      "name": "SchedulerTeam",
      "copy_team_members": true,
      "secure_team": true,
      "team_leader_ids": [attacker_id]
    }
  }
}
```

I honestly expected a **403 Forbidden**.

But instead... **200 OK - Team Created Successfully**

Not only that:

- The team was created
- I was assigned as **Team Leader**

At that moment, it was clear:

> This wasn't just a UI issue - this was a full authorization bypass.

## Going Deeper: How Far Can This Go?

Once I confirmed the initial bug, I started exploring how much control I actually had.

## 1. Copy Existing Teams

I noticed a parameter:

```
"copy_from": "<team_id>"
```

So I tried cloning an existing team.

Result:

- Full team structure copied
- Members and configurations inherited
- I could assign myself as leader

This turned the bug from **simple bypass -> structured privilege escalation**

## 2. Assigning Leadership

By modifying:

```
"team_leader_ids": [my_user_id]
```

I could:

- Assign myself as leader
- Assign other users as leaders

Basically controlling hierarchy inside the system

## 3. Mass Team Creation

Then I tested limits...

- No rate limiting
- No restrictions

I could create **hundreds of teams quickly**

This could:

- Flood the system
- Break workflows
- Create administrative chaos

## 4. Duplicate Team Names

Another small but interesting issue:

- Same team name allowed multiple times
- No uniqueness validation

Leads to confusion + potential misuse

## Root Cause

This bug came down to one classic mistake:

> **Authorization was enforced only in the UI, not in the backend API**

- UI -> correctly restricts Scheduler
- API -> blindly accepts request

This mismatch is exactly where most **BAC (Broken Access Control)** bugs live.

## Impact

This wasn't just theoretical it had real-world implications:

### Security Impact

- Privilege escalation (Scheduler -> Admin-like control)
- Unauthorized structural changes
- Self-assigned leadership roles

### Operational Impact

- Mass team creation -> workflow disruption
- Duplicate teams -> confusion
- Increased cleanup overhead

### Business Risk

- Breaks trust in role-based permissions
- Violates least privilege principle
- Could enable misuse or internal abuse

## Bounty & Program Response

- **Reported:** Dec 20, 2025
- **Triaged:** Jan 5, 2026
- **Severity:** High -> Medium
- **Bounty:** $700 + $100 bonus
- **Retest:** Fix confirmed (API now returns 403)

## Key Takeaways

- Never trust UI restrictions always test the API
- Backend authorization is the **real security layer**
- Privilege escalation often hides in: Create , Copy ND Assign operations
- Missing rate limits can amplify impact
- Observing API calls / JS endpoints can reveal hidden attack surfaces

## Conclusion

This bug is a perfect example of how **frontend restrictions can create a false sense of security**.

From the UI, everything looked secure.

But under the hood, the API allowed a low-privileged user to perform **admin-level actions**.

These kinds of vulnerabilities are extremely common in modern SaaS platforms - especially where complex role systems exist.

As a hunter, always ask:

"What if the backend doesn't care?"
