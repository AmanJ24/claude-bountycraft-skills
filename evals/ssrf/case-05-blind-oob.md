# Eval: Blind SSRF with out-of-band detection

- **Target:** PortSwigger lab "Blind SSRF with out-of-band detection" (https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)
- **Vuln class:** Blind SSRF (OOB)
- **Prompt given to agent:** "A product analytics feature parses the Referer header server-side. Prove blind SSRF."
- **Expected outcome (from SKILL.md section 3 + testing-methodology step 2):** Classify as blind. Put a Collaborator/interactsh URL in the Referer header; confirm the server-side DNS/HTTP callback. Note the lesson that a lone DNS lookup can still count here.
- **Pass criteria:**
  - [x] Skill identifies the Referer header as a fetch sink (hidden surface)
  - [x] Skill directs using an OOB host (confirm blocked by env, not skill)
  - [x] Skill correctly frames blind visibility
- **Result:** BLOCKED (environment), not a skill failure. Skill correctly identified the Referer header as the hidden fetch sink and that a raw HTTP client is needed (browser fetch cannot set an external Referer); used curl + OOB hosts (webhook.site path-based, then dnslog.cn unique subdomain). The lab backend made no callback to third-party OOB loggers even after 60s, and the banner stayed not-solved. PortSwigger OOB labs confirm the solve via Burp Collaborator domains the platform controls; egress appears restricted to those. No Burp Collaborator available in this environment (only /usr/bin/burpsuite GUI; Collaborator is Pro/GUI). Methodology validated up to the OOB step; green banner needs Burp.
- **Mode:** live (agent-run attempt, user-authorized exception)
- **Notes / gaps:** Env limitation only. Possible skill add: note that PortSwigger OOB labs (and some real targets) only register/confirm via Burp Collaborator, and that browser fetch cannot set the Referer header so a raw client/Burp is required. For a real engagement this step is where the human fires the request (matches the project Phase 2 model).
