---
title: "Privilege Escalation in Edit and Create Secret Endpoints Leads to Unauthorized Secret Modification"
author: 0xsom3a
url: https://hackerone.com/reports/3103755
date_published: 2025-04-22
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/3103755
bounty: none (Bounty: None)
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #3103755 | Reported to Dust | Status: Resolved | Severity: High (7.1) (initially high 8.2) | Weakness: Improper Access Control - Generic | Bounty: None | Disclosed: April 24, 2025

## Report (0xsom3a, April 22, 2025, 6:51am UTC)

# Summary

A user with the **Builder** role, a role that is **not expected** to manage secrets, can:

- List all existing secret names in the workspace.
- Create new secrets.
- Overwrite existing secrets simply by using the same name.

This behavior violates permission boundaries and leads to privilege escalation, tampering with app configurations, or unauthorized access to sensitive data.

# Steps to Reproduce

## Step-1: Get All Secret Names in the Workspace

As a Builder, send a `GET` request to the secrets endpoint to enumerate all existing secret names.

```
GET /api/w/[workspace_id]/dust_app_secrets HTTP/2
```

This returns a list of secrets with their `id`, `name`, `created_at`, etc., but without showing the secret `value`.

```
{
"secrets":[
{"name":"NAME-1","value":"•••••••"},
{"name":"NAME-2","value":"•••••••"}
]
}
```

## Step-2: Create or Overwrite a Secret

Now, send a `POST` request to create a new secret.

```
POST /api/w/[workspace_id]/dust_app_secrets HTTP/2
Host: dust.tt
Content-Type: application/json
Cookie:[appSession]

{
"name":"NAME-1",
"value":"malicious-value"
}
```

#### Behavior:

- If the `name` used in the request already exists in the workspace (as returned from step 1), the system will overwrite the existing secret's value.
- If the `name` is new, a new secret will be created.
- No error or warning is shown, overwrite happens silently.

# POC Video: [redacted]

# Expected Behavior

The Builder role should:

- Not be able to access the list of secret names.
- Not be able to create or update any secrets.

# Suggested Fix

- Enforce strict permission checks on all secret-related endpoints.
- Ensure only users with elevated roles (e.g., Admin, Owner) can list, create, or update secrets.

## Impact

This vulnerability allows users with the Builder role to:

- Discover all secret names in the workspace.
- Tamper with or overwrite secrets used by other users or apps.
- Create new secrets and potentially trick other users into using them.
- Escalate privileges by modifying secrets used in sensitive flows (e.g., API keys, tokens, credentials).

This could lead to: configuration manipulation, account compromise, supply chain attacks on internal tooling, loss of integrity of secret data.

## Timeline / comments

- Apr 22, 2025 6:52am: severity updated from high(8.2) to high(7.1).
- Apr 22, 2025 7:59am (spolu-dust, Dust staff): Triaged. "Thanks for your investigation. It is indeed not expected for the builder to add/modify/delete secrets. It is expected for them to be able to visualize them. Remediation happening here: https://github.com/dust-tt/dust/issues/12144"
- Apr 22, 2025 12:03pm: Resolved. "this is now fixed... admin can create edit remove secrets; builder can see the list of secrets but can't edit/remove/add; user can't see any of these"
- Apr 22, 2025: 0xsom3a requested disclosure; asked that the PoC video be redacted. Apr 24, 2025: Dust agreed; disclosed.
- Oct 12, 2025: 0xsom3a asked whether Dust can assist with requesting a CVE identifier.
