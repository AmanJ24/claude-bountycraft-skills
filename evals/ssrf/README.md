# SSRF evals

Test cases that validate whether `skills/ssrf/SKILL.md` actually helps find/confirm SSRF.
Each case maps a known-vulnerable target (PortSwigger Web Security Academy labs) to the
skill section(s) that should lead an agent to the solution.

Two validation modes:
- **Paper (default, no requests sent):** given only the lab description, does the skill's
  methodology name the right surface, test, and bypass that matches the lab's official
  solution? Graded by comparing the skill-derived plan to the known solution. Honors the
  project's "research phase only" rule: nothing is fired at a target.
- **Live (human-run, optional):** the human runs the lab in Burp and follows the skill;
  pass = lab solved. PortSwigger labs are authorized, self-owned training targets, but
  this still involves sending requests, so it is a human action, logged here, not agent-run.

Status: cases drafted 2026-11-16. Mode to run chosen with the user.
