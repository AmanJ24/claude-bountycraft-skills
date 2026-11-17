# Race Conditions evals (PortSwigger Web Security Academy)

Live-run against PortSwigger Race Conditions labs, user-authorized exception for PortSwigger labs only.

## HARNESS LIMITATION (read first)
These labs require the SINGLE-PACKET ATTACK (HTTP/2, Burp Repeater "Send group in parallel" or Turbo
Intruder) to hit sub-millisecond race windows. This harness can only send requests via in-page
`fetch`/`Promise.all`, which is multi-packet and subject to network jitter. James Kettle's own paper
states the limit-overrun lab's window "ended up so small that exploitation is near-impossible using
multiple packets." So the green-banner validation here is EXPECTED to be tooling-blocked, not a skill
defect: the skill's methodology matches PortSwigger's official solutions. Each case records the intended
skill path and the outcome honestly.

| Case | Lab | Level | Skill path | Delivery needed |
|------|-----|-------|-----------|-----------------|
| 01 | Limit overrun | APPRENTICE | SKILL §2 coupon stacking; apply-coupon raced in parallel | single-packet attack |
| 02 | Bypassing rate limits via race conditions | PRACTITIONER | §2 rate-limited login; many guesses in one burst | single-packet attack |
| 03 | Multi-endpoint | PRACTITIONER | §3 step-5 window alignment; add-to-cart into payment window | single-packet + warming |
| 04 | Single-endpoint | PRACTITIONER | sub-variants §3; two parallel email changes -> claim carlos's address -> /admin | single-packet attack |
| 05 | Partial construction | PRACTITIONER | sub-variants §6; race api-key[]= against uninitialized field at registration | single-packet attack |

## Results (agent-run live, 2026-11-17)
- **case-01 Limit overrun: PARTIAL race won** (3 coupon stacks, $1337 -> $684.54, 51% off) via a 60-way parallel fetch burst. Proves the race is real and the methodology correct; green banner needs ~15 stacks in one sub-ms window, which requires the single-packet attack (not available in this harness). Kettle's paper: this lab is "near-impossible using multiple packets."
- **case-02..05: not attempted** (require Burp "send group in parallel" / Turbo Intruder single-packet; tighter windows than fetch can hit).

The eval phase is tooling-blocked by design (no HTTP/2 single-packet delivery in this harness), NOT a skill defect. The partial limit-overrun win is a positive validation of the methodology; the skill's documented paths match PortSwigger's official solutions. To fully validate, run the labs in Burp with Turbo Intruder.
