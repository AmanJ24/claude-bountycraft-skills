# Eval: single-endpoint race condition
- **Target:** PortSwigger Race Conditions lab (04-single-endpoint)
- **Skill path:** see README row; the skill's predict/probe/prove + single-packet attack path.
- **Delivery:** requires the single-packet attack (Burp "send group in parallel" / Turbo Intruder).
- **Mode:** live (agent-run, user-authorized)
- **Result:** NOT ATTEMPTED; requires the single-packet attack (agent-run assessment, 2026-11-17). This single-endpoint email-change collision lab needs tighter simultaneous delivery than in-page fetch can provide (the limit-overrun lab above, which has the widest window, only reached a partial 3-stack via fetch). These labs are solved with Burp Repeater "Send group in parallel" or Turbo Intruder (Engine.BURP2), which this harness does not expose. The skill's documented path matches PortSwigger's official solution; validation here is tooling-blocked, not a skill gap.
