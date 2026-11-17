# Exploiting client-side prototype pollution in the wild (s1r1us)

Real-world research with multiple disclosed findings. A large collaborative effort scanning disclosure programs for client-side prototype pollution (CSPP) and chaining it to XSS via script gadgets. Conceptual pollute -> gadget -> sink framing, with named targets and bounties.

- **Vuln type / sub-variant:** Client-side prototype pollution escalated to XSS (and account takeover) through script gadgets. Two detection cases: (1) attacker-controlled `location.hash`/`location.search` parsed into an object, and (2) client-side merge of user-influenced JSON (often self-XSS, sometimes upgradable to reflected).
- **Target type:** JS-heavy web apps using query/hash parsers or object-merge utilities (canjs-deparam, jquery-deparam, backbone query params, component querystring) and any library whose code reads a property that can be polluted (jQuery, SwiftType, knockout, etc.).
- **Recon signal:** Appending prototype-pollution probes to the URL and checking whether `Object.prototype` gains the property. Roughly 80% of nested parameter parsers were found vulnerable. Automated with a Selenium bot over subdomains and a Chrome extension (PPScan) that scans endpoints as the user browses; CodeQL used for the JSON-merge (case 2) class.
- **Root cause (pollute -> gadget -> sink, conceptual):**
  1. Pollute: a parser or merge walks keys like `__proto__[x]`, `__proto__.x`, `constructor.prototype.x`, or `x[__proto__][y]` and assigns onto `Object.prototype`, so the injected property becomes visible on every object.
  2. Gadget: somewhere in the app's libraries a code path reads a property that is normally absent and uses it unsafely. Because the prototype now supplies that property, the attacker value flows into the gadget. Examples cited: a library that iterates config hooks and `eval`s a polluted value; jQuery reading `delegateTarget`/`handleObj` and passing a polluted HTML string into a jQuery call; polluted `wrapMap` entries landing in an `innerHTML` sink.
  3. Sink: the polluted value reaches an execution sink (`eval`, `innerHTML`/`srcdoc`, dynamically created script/worker, or an injected event-handler attribute such as `onerror`/`onmousemove`/`onanimationstart`), producing XSS.
- **Pollution sources used:** nested query/hash parsers (canjs-deparam, jquery-deparam, backbone, component querystring), and client-side merge of attacker-saved JSON.
- **Gadget sinks used:** `eval` of a polluted config value, jQuery DOM-manipulation reading polluted properties, `innerHTML`/`srcdoc`, polluted `script`/`img` `src`+`onerror`, polluted inline event handlers, and Web Worker blob URLs (low impact alone).
- **Exploitation steps / named targets:**
  1. Case Study 1 (CodeQL): found a vulnerable merge via CodeQL, polluted the prototype by POSTing a JSON payload the client later reparsed, found a proper gadget after Web Worker dead-end, then escalated self-XSS to account takeover by loading sensitive pages in iframes and reading SAML-authenticated data. Bounty $4000.
  2. Jira Service Management (<4.18.0): backbone query params polluted; posix gadget using polluted `onmousemove`/`isFresh` and several untrusted-types gadgets. Reported to Jira ($600). Fix blacklisted `__proto__`/`constructor`/`prototype` but stripped `[]` from keys, so BlackFan bypassed with `__pro[]to__`.
  3. jira.mozilla.com: same Jira issue, Mozilla awarded $2000.
  4. apple.com (/shop/buy-watch/apple-watch): canjs-deparam pollution found via the Chrome extension; gadget `__proto__[src]=image&__proto__[onerror]=alert(1)`. Apple's first fix (checking for literal `__proto__`) was bypassed via `[constructor][prototype]`; finally fixed by removing location parsing. Reflected XSS, $6000.
  5. HubSpot Analytics (deparam): widely embedded third-party; multiple fix bypasses (constructor-chain, Nikita Stupin's `?__proto__=&0[taint]=polluted`) before a null-object fix.
  6. Segment Analytics (component querystring): pollution only via numeric keys (`__proto__[123]=...`); no gadget found, so no XSS impact (illustrates pollution without a usable sink).
- **Bypasses used:** `[]`-stripping bypass (`__pro[]to__`), `constructor.prototype` chain to dodge `__proto__` blacklists, numeric-key pollution, case-based and null-object fix evasions.
- **Impact:** Reflected XSS and account takeover across major targets (Apple, Mozilla, Jira, HubSpot-embedded sites); ~80 bugs reported, 18 vulnerable libraries catalogued.
- **Bounty:** $4000 (case study 1), $2000 (Mozilla), $600 (Jira), $6000 (Apple reflected XSS); others out of scope / unpaid.
- **Lesson / pattern:** CSPP is only impactful when a gadget bridges the polluted prototype to a sink; hunt both halves. Deny-list `__proto__`/`prototype`/`constructor` immediately before assignment (and handle key normalization so `[]`/encoding tricks cannot slip through), prefer null-prototype objects, consider `Object.freeze(Object.prototype)` and Node's `--disable-proto`.
- **Source:**
  - Title: A tale of making internet pollution free - Exploiting Client-Side Prototype Pollution in the wild
  - Author: Mohan Sri Rama Krishna P (s1r1us) with research team
  - URL: https://blog.s1r1us.ninja/research/PP
  - Date: not stated (circa 2020-2021)
  - Disclosed report: references https://hackerone.com/reports/986386 (wcbowling) as prior art
  - Raw file: `sources/xss/28-s1r1us-client-side-prototype-pollution.md`

Tags: prototype-pollution, gadget, account-takeover, methodology, filter-bypass
