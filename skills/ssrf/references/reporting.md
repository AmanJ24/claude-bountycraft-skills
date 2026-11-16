# Reporting SSRF

What made reports in this set land at high severity and high trust. Write up the *impact* and the *root cause*, not just the callback.

## Template

**Title:** `SSRF in <feature> via <mechanism> leading to <impact>`
e.g. "SSRF in webhook delivery via 303 redirect leaking AWS role credentials".

**Summary (2 to 3 sentences):** where the fetch happens, what you made it reach, and the concrete impact (credentials read / internal service reached / file read).

**Steps to reproduce:**
1. Exact request(s), including the feature path and the parameter.
2. The bypass string, verbatim, if a filter was involved (redirect status code, IPv6/encoded IP, rebinding host, Unicode form).
3. Your redirector/listener behavior if used (status code, Location header).
4. Where the response surfaced (delivery log, generated PDF/PNG, import content, test output).

**Proof of impact:** the minimum evidence. One credential with the secret value **redacted**, one internal service banner/name, the first line of one file, or a screenshot of the metadata response. State the response class (full / partial / blind).

**Root cause:** name it precisely so triage can fix it:
- "Redirects followed without re-validating the target against internal ranges; only 303 was handled."
- "Blocklist compared input text, not the resolved/post-redirect destination."
- "Validator used Ruby Resolv while the HTTP client used the OS resolver; they disagree on `0177.1`."
- "Upload processed before the content-type check."

**Impact and risk:** translate to business risk. Metadata creds -> cloud account access and data; internal port scan -> attack surface; file read -> secrets; chained -> infrastructure/RCE. Suggest a CVSS with justification.

**Remediation:** resolve the host once and validate the *resolved* IP against internal ranges with the same resolver the client uses; re-validate every redirect hop; fail closed on empty/failed resolution; disable scripts and restrict egress in renderers; block link-local/metadata at the network; prefer an allowlist of exact external hosts where feasible.

**Scope note:** confirm the finding is in scope and state that you stopped at proof.

## What raised trust and severity (observed)
- **Leading with demonstrated impact.** The $25k and $5k findings showed a credential or clear internal reach, not just "the server called my host" (notes 01, 20, 03, 18).
- **Explicitly stopping at proof and asking before deeper escalation.** Raised trust and rating in the Matrix ($6k) and HackerOne analytics ($25k) reports (notes 02, 01). Say what you did *not* do and why.
- **Exact bypass + root cause.** Lets triage reproduce despite timing noise; several reports were initially hard to reproduce (resolver/timing) and the clear root cause carried them (notes 22, 16, 09).
- **Fingerprinting the engine** from the callback User-Agent to explain the mechanism (notes 07, 27).

## What to avoid
- Reporting a bare OOB callback as if it were the impact. If you cannot show internal reach, a readable response, or a dangerous protocol, it is likely low/informational (see false positives in `../SKILL.md`).
- Dumping the whole internal environment. One concrete proof per claim; do not move laterally.
- Including live secret values, customer data, or the full contents of files you read. Redact.
- Claiming SSRF when the request was client-side (your browser/IP/cookies). Prove server origin first.
