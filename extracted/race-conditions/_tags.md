# Canonical tags (Race Conditions)

Normalized 2026-11-17. Use these spellings in `Tags:` lines.

- Sub-variant: limit-overrun, multi-endpoint, single-endpoint, partial-construction, time-sensitive, rate-limit-bypass, otp-brute, double-spend, social-action, auth-flow, state-corruption, toctou, points-inflation
- Technique: single-packet-attack, last-byte-sync, first-sequence-sync, turbo-intruder, connection-warming, send-group-parallel, benchmarking, predict-probe-prove, sub-state
- Mechanism: check-then-act, non-atomic, missing-lock, shared-key-collision
- Impact: financial-loss, free-services, account-takeover, privilege-escalation, data-exposure, unlimited-redemption
- Meta: methodology, taxonomy, reference, tooling, defense

Notes:
- `check-then-act` + `non-atomic` are the near-universal root-cause tags (the read-then-write gap).
- `single-packet-attack` is the default modern technique (HTTP/2); `last-byte-sync` is the HTTP/1.1 fallback; `first-sequence-sync` (note 04) is the expert escalation for ~10k requests.
- Normalized the stray `technique` tag (note 02) to `methodology`.
- References/technique sources: 01-09 (minus findings), 18, 20 carry methodology/taxonomy/tooling.
- Real findings: 10-17 (H1). 18 is a practitioner lab walkthrough (Tesla/Uber examples are the author's uncited claims, flagged).
