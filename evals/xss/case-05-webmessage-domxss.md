# Eval: DOM XSS using web messages (postMessage) [PRACTITIONER]
- **Target:** /web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages
- **Sub-variant:** postMessage source -> sink (innerHTML/eval); exploit-server deliver-to-victim
- **Skill path:** advanced-techniques.md postMessage: host an iframe that postMessages a payload the listener passes to a sink; deliver via exploit server to the victim (server-side solve, no dialog in agent tab).
- **Pass:** [x] skill identifies the message listener/sink; [x] crafts the postMessage exploit; [x] delivered -> lab solved
- **Mode:** live (agent-run, user-authorized)
- **Result:** PASS (agent-run live, 2026-11-17). Confirmed sink `<div id="ads">` fed by `innerHTML = event.data` with no origin check (skill's postMessage failure-mode "no check"). Proved the sink locally first with a benign DOM-write marker (`<img src=1 onerror="document.title=...">` fired, title changed), then built the exploit-server page per the skill: `<iframe src=LAB onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">`. Delivered to victim (server-side, no dialog in agent tab) -> green banner "Congratulations, you solved the lab!", status is-solved. Clean full pass, no automation caveat (exploit-server path avoids the alert-dialog limit).
