---
title: "$1,500 Bounty: Business Logic Flaw in Project Secrets Management"
author: Abhi Sharma (@a13h1)
url: https://infosecwriteups.com/1-500-bounty-business-logic-flaw-in-project-secrets-management-4b985175071e
date_published: 2025-09-19
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: $1,500
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. Medium nav/footer chrome trimmed. -->

# $1,500 Bounty: Business Logic Flaw in Project Secrets Management
By Abhi Sharma, Sep 2025.

A Business Logic Vulnerability in a SaaS platform that allowed Editor-level users to bypass UI restrictions and directly add secrets to projects, something only Owners were supposed to do. Awarded $1,500.

## Background
On the platform:
- Owners -> full permissions (manage projects, secrets, configurations)
- Editors -> edit project code/resources, but should not manage sensitive project secrets
- Viewers -> read-only.

Additionally, on free-tier plans, the Manage section (where secrets are handled) is not available.

So logically, an Editor on a free-tier project should never be able to create or update secrets. But the API told a different story.

## The Vulnerability
Even though the UI prevented Editors from accessing the Manage section, the backend API still allowed Editors to add secrets directly with a crafted request.

## Proof-of-Concept Request
POST /api/v1/secrets/namespace/project:<project_id>/name/<secret_name> HTTP/2
Host: target.com
Content-Type: application/json

{"value":"csdc"}

- Replace <project_id> with the project UUID.
- Replace <secret_name> with any arbitrary key.
- Forward the request.

## Response
HTTP/2 200 OK
{ "message": "Secret created successfully", "name": "<secret_name>", "value": "csdc" }

The secret was created under the project namespace.

## Steps to Reproduce
1. Create or access a free-tier project where you are assigned as an Editor.
2. Observe that the Manage section is hidden in the UI.
3. Intercept traffic with Burp Suite and send a crafted API request (like the one above).
4. Server accepts the request -> secret successfully added.

## Expected Behavior
The API should enforce the same restrictions as the UI: Editors should not be able to create secrets; free-tier projects should not allow secret management at all.

## Actual Behavior
- UI restriction existed
- API restriction was missing
- Editors could add or alter secrets despite policy restrictions.

## Root Cause
- Mismatch between UI and backend authorization.
- UI correctly hid the Manage section for Editors / free-tier users.
- API endpoint trusted role metadata incorrectly and allowed requests.
- Lack of server-side enforcement -> client-side control bypass.

## Impact
- Privilege Escalation -> Editors could perform Owner-only actions.
- Policy Violation -> free-tier limitations bypassed.
- Sensitive Data Risks -> malicious editors could inject secrets affecting deployments, pipelines, or configurations.
- Misleading UX -> UI indicated Editors cannot manage secrets, but API contradicted it.

## Timeline
- July 5, 2025 - report submitted (Severity: High, CVSS 8.3).
- July 9, 2025 - triaged.
- July 10, 2025 - fix confirmed + $1,500 bounty awarded.

## Final Thoughts (author)
A Business Logic Vulnerability: no injection payloads, no fuzzing, just careful observation and testing for policy mismatches between UI and API.
