# Eval: Reflected XSS into a template literal (Unicode-escaped) [PRACTITIONER]
- **Target:** /web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped
- **Sub-variant:** JS template-literal context; angle brackets/quotes/backslash/backticks escaped
- **Skill path:** contexts-and-payloads.md JS template-literal context: inject `${...}` inside the backtick template, e.g. search term -> `${alert(document.cookie)}` (no breakout chars needed since we are already inside the literal).
- **Pass:** [x] skill identifies template-literal context; [x] uses ${} without needing blocked chars; [x] executes
- **Mode:** live (agent-run, user-authorized)
- **Result:** PASS (agent-run live, 2026-11-16). Confirmed context: reflection inside a backtick template after a quote; ${1} survived unescaped. Delivered /?search=${alert(document.domain)}; alert fired, lab solved (verified from a second tab since the alert dialog blocks the triggering tab).
