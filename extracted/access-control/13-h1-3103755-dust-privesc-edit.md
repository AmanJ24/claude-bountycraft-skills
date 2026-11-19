# Dust: Builder role can list, create and overwrite workspace secrets

- **Vuln type / sub-variant:** Vertical privilege escalation via missing role check on a secrets management API (read/create/overwrite by under-privileged role)
- **Target type:** SaaS workspace REST API (per-workspace app secrets endpoint, cookie session)
- **Recon signal:** The product has a tiered role model (Admin/Owner, Builder, User). The reporter, logged in as Builder, tried the secrets endpoints directly, a feature that role is not meant to manage.
- **Root cause:** The `dust_app_secrets` endpoint did not enforce a role check for list and write operations, so any member with Builder privileges passed authorization.
- **Exploitation steps:**
  1. Log in as a Builder in a workspace.
  2. Send `GET /api/w/<workspace_id>/dust_app_secrets` and receive every secret name (values masked).
  3. Send `POST /api/w/<workspace_id>/dust_app_secrets` with JSON `{"name":"<existing or new name>","value":"<attacker value>"}` and the session cookie.
  4. Observe that reusing an existing name silently overwrites the stored value, and a new name creates a secret, with no error or warning.
- **Bypasses used:** None needed. Plain direct API call as the lower role; same name overwrites without a conflict check.
- **Impact:** Secret name enumeration, tampering with secrets consumed by other users' apps (API keys, tokens), planting attacker-controlled secrets, integrity loss and possible downstream compromise. Severity High (7.1).
- **Bounty:** None (resolved, disclosed)
- **Lesson / pattern:** When a product has more than two roles, replay each privileged endpoint as every intermediate role, not just as an anonymous or lowest user. Intermediate roles often get "read" and "write" conflated, and a PUT/POST-by-name upsert makes overwrite trivial. Triage clarified intended design: Builder may view the list but not mutate, User sees nothing.
- **Source:**
  - Title: Privilege Escalation in Edit and Create Secret Endpoints Leads to Unauthorized Secret Modification
  - Author: 0xsom3a
  - URL: https://hackerone.com/reports/3103755
  - Date: 2025-04-22
  - Disclosed report: https://hackerone.com/reports/3103755
  - Raw file: `sources/access-control/13-h1-3103755-dust-privesc-edit.md`

Tags: vertical-privesc, privilege-escalation, missing-server-check, mflac
