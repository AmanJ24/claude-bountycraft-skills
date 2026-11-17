# Client-side prototype pollution to DOM XSS (Web Security Academy)

- **Vuln type / sub-variant:** Client-side prototype pollution (source) chained to a gadget that reaches an XSS sink. Technique page, scraped in full.
- **Target type:** JS front ends and third-party libraries that merge/parse URL, hash or JSON input into objects (query-string parsers, deep-merge helpers, analytics/tag libraries).
- **Recon signal:** Any client code that turns `location.search`, `location.hash` or JSON into nested objects; DOM Invader flags sources and gadgets automatically.
- **Root cause:** Unsafe recursive merge/assignment lets attacker keys such as `__proto__` write onto `Object.prototype`, and later code reads an undefined property that now resolves to the polluted value and passes it to a sink.
- **Exploitation steps:**
  1. Find a source by injecting an arbitrary property: `vulnerable-website.com/?__proto__[foo]=bar`; also try the URL fragment and JSON input.
  2. Confirm in the console: `Object.prototype.foo` returns `"bar"` if polluted, `undefined` if not.
  3. If nothing sticks, switch notation: `?__proto__.foo=bar`.
  4. Find a gadget: list properties the app or its libraries read, then debug: pause the script early and run `Object.defineProperty(Object.prototype, 'YOUR-PROPERTY', { get(){ console.trace(); return 'polluted'; } })`; a stack trace shows where the property is read, then step to see if it reaches `innerHTML`, `eval()` or a script `src`.
  5. Combine source and gadget in one URL, e.g. `?__proto__[<gadgetProp>]=<payload>`, to reach the sink (labs: DOM XSS via client-side prototype pollution, via an alternative vector).
- **Bypasses used:**
  - Constructor route when `__proto__` is stripped: reach the prototype through `obj.constructor.prototype` (equivalent to `__proto__`); the page describes this route in prose, so a query form like `?constructor[prototype][foo]=bar` is an inference to test, not a quoted payload.
  - Non-recursive key stripping: `?__pro__proto__to__.gadget=payload` collapses to `__proto__.gadget`.
  - Gadgets in third-party libraries found with DOM Invader; PortSwigger also documents pollution gadgets in browser APIs for bypassing weak defences.
- **Impact:** DOM XSS (arbitrary JS) via the gadget sink; works from a crafted link.
- **Bounty:** n/a
- **Lesson / pattern:** Pollution is two bugs: a source that writes `Object.prototype` and a gadget that reads an unset property into a sink. Test both, and do not assume `__proto__` filtering is complete.
- **Source:**
  - Title: Client-side prototype pollution vulnerabilities
  - Author: PortSwigger (Web Security Academy)
  - URL: https://portswigger.net/web-security/prototype-pollution/client-side
  - Date: not stated (scraped 2026-11-16)
  - Disclosed report: none
  - Raw file: `sources/xss/19-portswigger-client-side-prototype-pollution.md`

Tags: dom, prototype-pollution, gadget, filter-bypass, methodology
