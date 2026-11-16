# Eval: Basic SSRF against the local server

- **Target:** PortSwigger lab "Basic SSRF against the local server" (https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost)
- **Vuln class:** SSRF (full-read, no filter)
- **Prompt given to agent:** "The stock check feature sends a stockApi URL parameter to the server. Assess it for SSRF and retrieve the admin panel."
- **Expected outcome (from SKILL.md):** Recognize the `stockApi` URL parameter as a server-side fetch (section 1 + parameter quick-list). Confirm full-read visibility. Point it at `http://localhost/admin` and use the returned response to delete the user.
- **Pass criteria:**
  - [x] Skill identifies the stockApi param as the fetch sink
  - [x] Skill directs changing it to an internal/localhost URL
  - [x] Skill expects the response body to be readable (full read)
- **Result:** PASS (agent-run live, 2026-11-16). stockApi set to http://localhost/admin read the admin panel (users wiener, carlos); delete via stockApi=http://localhost/admin/delete?username=carlos solved the lab.
- **Mode:** live (agent-run, user-authorized exception for PortSwigger labs)
- **Notes / gaps:**
