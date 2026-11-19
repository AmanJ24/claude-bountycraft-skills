# Access Control evals (PortSwigger Web Security Academy)

Live-run plan against PortSwigger Access Control labs, user-authorized exception for PortSwigger labs only. Unlike the race-conditions evals, these labs are each a single ordinary request, so they are fully driveable with in-page `fetch` or browser navigation: no single-packet attack or Burp required. The eval is expected to PASS when the browser extension is connected.

## Case selection (spans the skill taxonomy, excludes pure IDOR -> idor skill)

| Case | Lab | Level | Skill path (SKILL.md / reference) | Intended solve |
|------|-----|-------|-----------------------------------|----------------|
| 01 | Unprotected admin functionality | APPRENTICE | §1 hidden/role-gated route reachable directly; recognizing-and-surfaces #3 | Read `/robots.txt`, force-browse `/administrator-panel`, delete carlos |
| 02 | User role controlled by request parameter | APPRENTICE | §1 client-supplied privilege; recognizing-and-surfaces #4 / testing-methodology §2 | Set `Admin=true` cookie (or `roleid=2` in profile body), reach `/admin`, delete carlos |
| 03 | URL-based access control can be circumvented | PRACTITIONER | §4 header bypass; 403-401-bypass §2 (X-Original-URL) | `GET /` + `X-Original-URL: /admin/delete?username=carlos` |
| 04 | Method-based access control can be circumvented | PRACTITIONER | §4 method fuzzing; 403-401-bypass §1 | Capture admin role-change POST, replay as low-priv with method/endpoint tweak |
| 05 | Multi-step process with no access control on one step | PRACTITIONER | §3 force-browse a step; testing-methodology §4 | Submit the final admin role-upgrade step directly as a low-priv user |

Each case validates a distinct branch of the skill: force-browsing, client-supplied role, header bypass (the 403 arsenal), method bypass, and multi-step gap.

## Results (agent-run live, 2026-11-19): 5/5 PASS

After the user reconnected the browser extension, all five cases were driven end to end via `navigate` + in-page `fetch` and reached the green "LAB Solved" banner (`.widgetcontainer-lab-status.is-solved`).

| Case | Lab | Level | Result | Skill branch validated |
|------|-----|-------|--------|------------------------|
| 01 | Unprotected admin functionality | APPRENTICE | **PASS** | force-browse hidden route (robots.txt -> `/administrator-panel`) |
| 02 | User role can be modified in user profile* | APPRENTICE | **PASS** | client-supplied role (`roleid:2` mass-assignment in profile body) |
| 03 | URL-based access control can be circumvented | PRACTITIONER | **PASS** | header bypass (`X-Original-URL: /admin`), validates Clario note 10 |
| 04 | Method-based access control can be circumvented | PRACTITIONER | **PASS** | method fuzzing (POST 401 -> GET 200) |
| 05 | Multi-step process with no access control on one step | PRACTITIONER | **PASS** | force-browse the final commit step as low-priv |

\* case-02 was swapped from the originally-planned cookie-based "User role controlled by request parameter" lab to its body-based sibling, because the harness cannot overwrite the lab's HttpOnly `Admin` cookie via JS (`document.cookie` write is ignored for that cookie; `fetch` cannot set a `Cookie` header). The sibling lab exercises the identical skill branch (client-supplied/self-assigned role) through a JSON body. See case-02 file.

**No skill gaps surfaced.** Every case was solved directly from the skill's documented methodology (SKILL.md §1/§3/§4, references/403-401-bypass.md, references/testing-methodology.md). This matches the skill's definition-of-done (at least one eval passes); access-control labs need no special tooling, so unlike race-conditions there was no harness ceiling here apart from the one cookie-write quirk (worked around, documented).

### Harness notes (for future runs)
- In-page `fetch` can set custom headers (`X-Original-URL`) and arbitrary methods, so header/method/URL/body-based access-control labs are fully driveable.
- It CANNOT set the `Cookie` header or overwrite an HttpOnly cookie, so cookie-value access-control labs need Burp.
- Some bypasses mutate state while returning a non-2xx status (case-03 delete returned 403 but still deleted carlos): confirm the *effect*, not just the response code.
