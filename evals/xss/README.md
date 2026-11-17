# XSS evals (expert-level validation)

Live-run (agent) against PortSwigger Web Security Academy labs, user-authorized exception for
PortSwigger labs only. Chosen to span Practitioner to EXPERT, covering the skill's hard variants.
Benign markers only (alert/print/DOM-write); secrets never exfiltrated to real hosts.

Note on dialogs: PortSwigger marks "alert(...)" labs solved when the hooked alert runs. A native
alert dialog blocks the automated tab, so where needed the solve is verified from a second tab /
via lab status fetch, or via the exploit server's Deliver-to-victim (server-side, no dialog). Any
lab that genuinely needs an interactive-only step is documented, not faked.
