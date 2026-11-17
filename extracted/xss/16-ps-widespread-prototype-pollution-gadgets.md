# Widespread prototype pollution gadgets (browser APIs + common libraries)

- **Vuln type / sub-variant:** Client-side prototype pollution (CSPP) gadgets that turn a pollution source into DOM XSS or property hijack. Catalogues gadgets in browser JS APIs and in popular third-party scripts.
- **Target type:** Library/app class: sites with a client-side prototype-pollution source (e.g. a merge/parse routine reachable via query string `__proto__[x]=...`) that also call browser APIs taking an options object, or load analytics/tag-manager scripts whose config properties are read from the polluted prototype.
- **Recon signal:** Tester recognizes exploitability in two parts. (1) A pollution source: a URL/JSON into a recursive merge/clone that lets `__proto__[prop]` set `Object.prototype.prop`. (2) A gadget: unguarded code that reads a property it never set, on an options object or config, and flows it to a sink. DOM Invader surfaces these by reporting a sink (`script.src`, `setTimeout`, `eval`, `innerHTML`) whose value came from a polluted property name plus canary.
- **Root cause (mechanism):** When an API or library reads an object property that the developer did not explicitly set, JS walks the prototype chain and returns the polluted `Object.prototype` value. So pollute -> gadget -> sink: the source writes `Object.prototype.X`; the gadget is code that uses `X` without a guard; the sink executes it. The source syntax is typically `__proto__[X]=payload` (URL-encoded `__proto__%5BX%5D=...`).
- **Exploitation steps (pollute -> gadget -> sink chains):**
  1. `fetch` options gadget: `Object.prototype.body="foo=bar"` then an app `fetch('/', {method:"POST"})` that omits `body` sends attacker-controlled body. Any omitted option (headers, method, body) is pollutable.
  2. `Object.defineProperty` value gadget: if a descriptor omits `value`, the engine reads `Object.prototype.value`, so `Object.prototype.value='overwritten'` overrides a property declared non-writable/non-configurable. Polluting `configurable` can re-open a locked property and aid sandbox escape.
  3. Storage getter gadget: `localStorage`/`sessionStorage` inherit from `Object.prototype`; code using property access (`localStorage.foo`) instead of `getItem()`/`get()` returns the polluted value.
  4. Google Analytics `hitCallback` -> `setTimeout` sink: GA runs `setTimeout(c,10)` where `c` is read from the (pollutable) `hitCallback`, giving DOM XSS on any site with a source.
  5. Google Tag Manager `sequence` -> `eval` sink (and `event_callback` -> `setTimeout`). Example source shape: `?__proto__[sequence]=alert(document.domain)-` (trailing `-` needed because the value lands in a JS expression beside an integer).
  6. Adobe Dynamic Tag Management gadgets: `cspNonce`/`bodyHiddenStyle` -> `innerHTML` sink; `trackingServerSecure` -> partial `script.src` control (host controllable) -> DOM XSS.
- **Bypasses used:** Gadget sinks chosen so the polluted value is trusted by the sink; expression-padding tricks (trailing `-`) to keep the injected value syntactically valid in its context.
- **Impact:** DOM XSS on real sites (a game site, a WordPress domain, others) whenever a CSPP source coexists with these very common analytics/tag-manager or API gadgets; property-hijack and possible sandbox-escape via `defineProperty`/`configurable`.
- **Bounty:** n/a (research). Google declined to fix, calling source-free code the customer's responsibility.
- **Lesson / pattern:** A prototype-pollution source is only half the bug; widespread gadgets (fetch/defineProperty/storage, GA, GTM, Adobe DTM) supply the rest, so these gadgets live in dependencies most sites already ship. Defenses: use null-prototype objects (`Object.create(null)`) for API option/config objects, always specify `value` in descriptors, read storage via `getItem`, and scan bundled libraries for gadgets. When hunting, prove a source then reach for a known gadget rather than needing a bespoke sink.
- **Source:**
  - Title: Widespread prototype pollution gadgets
  - Author: Gareth Heyes (PortSwigger Research)
  - URL: https://portswigger.net/research/widespread-prototype-pollution-gadgets
  - Date: 2022-06-22
  - Disclosed report: none (gadgets reported to Google; BlackFan corpus referenced)
  - Raw file: `sources/xss/16-ps-widespread-prototype-pollution-gadgets.md`

Tags: prototype-pollution, gadget, dom, csp-bypass
