# Rocket.Chat: guest to admin via role self-assignment and integration script

- **Vuln type / sub-variant:** vertical privilege escalation, chained role assignment (guest -> bot -> admin)
- **Target type:** chat platform with DDP/websocket methods and scriptable integrations
- **Recon signal:** a user-management method (`insertOrUpdateUser`) callable by ordinary users, plus a role (`bot`) holding a powerful permission (`manage-own-integrations`)
- **Root cause:** the method did not verify the caller may assign roles. Only a direct grant of `admin` was blocked, so granting an intermediate role bypassed it. Integration scripts then run with server-side access, allowing role changes.
- **Exploitation steps:**
  1. Log in as a guest and learn your own user `_id` from traffic.
  2. Send a websocket DDP method call to `insertOrUpdateUser` with your `_id` and `roles` set to `["user","bot"]`.
  3. With the bot role's integration permission, create a custom incoming integration whose script calls the server role helper to add your user to `admin`.
  4. Trigger the integration to become global admin.
- **Bypasses used:** deny-list on the target role (admin) sidestepped by self-assigning an adjacent role with a dangerous capability; script sandbox not isolated from server API
- **Impact:** guest becomes server administrator (Critical 9.9)
- **Bounty:** hidden
- **Lesson / pattern:** map which low roles can be self-assigned and what each grants; indirect escalation through a permission that executes code or edits roles beats a direct-admin check. Test websocket/RPC methods, not just HTTP.
- **Source:**
  - Title: Guest Privilege Escalation to admin group
  - Author: gronke
  - URL: https://hackerone.com/reports/501081
  - Date: 2019-02-25 (disclosed 2024-08-10)
  - Disclosed report: https://hackerone.com/reports/501081
  - Raw file: `sources/access-control/09-h1-501081-rocketchat-guest-to-admin.md`

Tags: vertical-privesc, mass-assignment-role, privilege-escalation, admin-access, missing-server-check, trust-user-input
