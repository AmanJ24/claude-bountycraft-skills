# PortSwigger taxonomy of business logic flaw categories and matching labs

- **Vuln type / sub-variant:** methodology/taxonomy (reference page, not a single finding)
- **Target type:** General web applications: shops, banking-style transfers, multi-step auth, any domain-specific workflow
- **Recon signal:** Not applicable to one bug. The page teaches where to look: client-side-only validation, numeric fields, mandatory parameters, multi-step flows, and price or discount adjustments driven by user actions.
- **Root cause:** Categories defined by the original developer mistake: (1) trusting client-side controls and assuming users only use the UI; (2) not handling unconventional input such as negative, huge, or oddly typed values and long strings; (3) flawed assumptions about behavior: trusted users staying trustworthy (inconsistent controls after login), mandatory parameters always present, and steps always taken in order; (4) domain-specific flaws such as discount thresholds not rechecked after cart edits; plus two extras, encryption oracles and email parser discrepancies.
- **Exploitation steps:**
  1. Tamper with data after the browser sends it; try values outside what legitimate users enter and observe limits, normalization, and transformations.
  2. Remove each parameter one at a time (name and value), and follow multi-step flows to the end since a change in one step can affect later ones.
  3. Use forced browsing: skip steps, repeat steps, go back, hit steps with different parameter sets on the same URL; read error messages from half-initialized state.
  4. For commerce features, understand when discounts are computed and try to reach a state that no longer matches the criteria (add items to pass a threshold, then remove them).
  5. Check whether user-controlled text is encrypted and returned, and whether another input accepts that ciphertext; check email-domain parsing for encoding tricks.
- **Bypasses used:** Burp tampering, parameter deletion, forced browsing out of sequence, negative or extreme numerics. Named lab patterns: negative-amount transfer (balance check passes because -1000 is below balance, funds flow backward), flawed state machine authentication bypass, insufficient workflow validation, inconsistent security controls, infinite money loop, 2FA simple bypass and broken logic, dual-use endpoint, encryption oracle auth bypass, email parser discrepancy (expert).
- **Impact:** n/a (reference)
- **Bounty:** n/a (reference)
- **Lesson / pattern:** Identify the developer's assumption, then violate it. If one form mishandles odd input, assume others do too. For unfamiliar domains, read documentation and learn the business, since obscure domains are less picked over.
- **Source:**
  - Title: Examples of business logic vulnerabilities | Web Security Academy
  - Author: PortSwigger Web Security Academy
  - URL: https://portswigger.net/web-security/logic-flaws/examples
  - Date: unknown
  - Disclosed report: none
  - Raw file: `sources/business-logic/23-portswigger-logic-flaws-examples.md`

Tags: methodology, taxonomy, reference, client-trust, parameter-tampering, negative-quantity, workflow-bypass, state-machine, discount-abuse, order-of-operations, assumption-flaw, otp-bypass
