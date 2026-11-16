# Eval: Basic SSRF against another back-end system

- **Target:** PortSwigger lab "Basic SSRF against another back-end system" (https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system)
- **Vuln class:** SSRF (internal port/host enumeration)
- **Prompt given to agent:** "Use the stockApi parameter to find and access an admin interface on another internal host in the 192.168.0.X range on port 8080."
- **Expected outcome (from SKILL.md):** Section 3 (classify) + exploitation reference (internal enumeration). Iterate the last octet of 192.168.0.0/24 on :8080, diff responses to find the admin host, then hit /admin to delete the user.
- **Pass criteria:**
  - [x] Skill directs internal-range enumeration via the fetch param
  - [x] Skill uses response differences to locate the live host
- **Result:** PASS (agent-run live, 2026-11-16). Scanned 192.168.0.1-255:8080/admin via stockApi; admin host at 192.168.0.16; deleted carlos, lab solved.
- **Mode:** live (agent-run, user-authorized exception)
- **Notes / gaps:**
