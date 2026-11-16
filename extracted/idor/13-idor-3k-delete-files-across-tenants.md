# Tenant admin can delete other tenants' files by UUID (GraphQL)

Source completeness: partial (writeup is brief; no request/mutation names, endpoint paths or exact steps are given)

- **Vuln type / sub-variant:** Cross-tenant IDOR on a destructive action, GUID/UUID object IDs, role-dependent (only the tenant admin role was affected), via a GraphQL endpoint.
- **Target type:** Multi-tenant file storage app with tenants, folders, files, invited users and tenant admins; GraphQL API.
- **Recon signal:** Basic roles were already protected: an invited user could not touch admin-only folders or another tenant's files. The hunter retried the same cross-tenant delete with a higher-privilege role (tenant admin), suspecting authorization was implemented per role path rather than uniformly.
- **Root cause:** The GraphQL operation that lets tenant admins act on files accepted a client-supplied file UUID and checked the caller's role but not that the file belonged to the caller's tenant, unlike the invited-user path which enforced it.
- **Exploitation steps:**
  1. Confirm as an invited user in tenant A that access to admin-only folders and to tenant B files is denied.
  2. Repeat as a tenant A admin, supplying a tenant B file's UUID to the delete action.
  3. The delete succeeds. (How the foreign UUID was obtained is not stated in source.)
- **Bypasses used:** Switching to a role whose code path skipped the tenant-ownership check.
- **Impact:** Destructive cross-tenant file deletion, potentially without audit trail.
- **Bounty:** $3,000
- **Lesson / pattern:** Re-run every cross-object test under each role; authorization bugs often live in the privileged path only.
- **Source:**
  - Title: The IDOR That Got Me My First $3K And Let Me Delete Files Across Tenants
  - Author: Sid_x95
  - URL: https://medium.com/@Sid_x95/the-idor-that-got-me-my-first-3k-and-let-me-delete-files-across-tenants-be5a65021832
  - Date: 2025-11-09
  - Disclosed report: none
  - Raw file: `sources/idor/13-idor-3k-delete-files-across-tenants.md`

Tags: cross-tenant, uuid, delete, graphql
