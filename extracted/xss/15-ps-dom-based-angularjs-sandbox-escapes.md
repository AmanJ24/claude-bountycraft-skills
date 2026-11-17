# DOM-based AngularJS sandbox escapes (orderBy filter context)

- **Vuln type / sub-variant:** DOM-based AngularJS sandbox escape, specifically escapes that fire inside the `orderBy` filter context (no `{{ }}`), where common escapes relying on `$eval`/`$$watchers` do not work.
- **Target type:** Library/app class: AngularJS 1.x sites that feed user-controllable input (e.g. `location.hash`) into an `orderBy` filter, or into an attribute like `ng-click="...|orderBy:'...'"`. The `orderBy` filter evaluates its argument string as an Angular expression.
- **Recon signal:** Tester recognizes exploitability when a DOM source (hash/query) flows into an `orderBy` filter argument, or when they can inject into an Angular attribute that ends in `|orderBy:'...'`. The distinguishing tell versus classic CSTI: expressions run without `{{ }}`, and `$eval`/`$$watchers` are absent, so escapes must be chosen for that context.
- **Root cause (mechanism):** `orderBy` evaluates a string as an Angular expression, giving an expression-execution sink reachable from DOM input. The sandbox (same `ensureSafeObject`/`ensureSafeMemberName` family as the CSTI post) is still just a rewriter. Heyes developed escapes that work in this restricted context by (a) corrupting Angular's own expression rewriter so a syntax error lets attacker JS into the generated function, or (b) reaching the `Function`/`Object` constructors via property-descriptor manipulation that Angular's name checks miss.
- **Exploitation steps (technique in plain steps):**
  1. Identify a DOM source reaching an `orderBy` filter argument (or an injectable `ng-click`/attribute with `orderBy`).
  2. Choose a version-matched escape (ranges matter; only some classic escapes survive the orderBy context):
     - 1.0.1 to 1.1.5 and >=1.6.0: the `constructor.constructor('alert(1)')()` approach still works.
     - 1.2.0 to 1.2.18: Jan Horn's `getOwnPropertyDescriptor` + generic `call.call` chain to reach the `Function` constructor.
     - 1.2.24 to 1.3.20 (Heyes): a minimal "rewriter break-out" that smuggles JS through an object-property string, e.g. the compact `{}.")));alert(1)//";` form, exploiting that Angular duplicates the property string into its rewritten code.
     - 1.4.x: `__lookupGetter__`/`event.target.defaultView` chain to reach `window`/`location`.
     - 1.5.0 to 1.5.8: nested `orderBy` as an eval primitive, e.g. override `charAt` with `[].join` then `[1]|orderBy:'x=alert(1)'`.
     - 1.5.11 and CSP mode (attribute context): `$event.path|orderBy:'...'` reaches an array holding `window`/`document`; under CSP call the function indirectly via `Array.from([1],alert)` to avoid the blocked direct call.
  3. The chosen escape runs JS outside the sandbox.
- **Bypasses used:** orderBy-as-eval, native-method override (`charAt`=`[].join`), array-accessor trick to dodge `ensureSafeMemberName` (`{}[['__proto__']]`), property-descriptor manipulation (`getOwnPropertyDescriptor`/`defineProperty`), `__lookupGetter__`/`$event.path` to reach `window`/`document`, and `Array.from` indirect call for CSP mode.
- **Impact:** XSS from a DOM source on Angular sites using `orderBy` with user input, including a CSP-mode bypass that works in an attribute context.
- **Bounty:** n/a (research).
- **Lesson / pattern:** `orderBy` (and any filter that evaluates a string) is an expression sink: never pass user input, including `location.hash`, into it. Escapes exist for every sandbox version and even for CSP mode, so assume the AngularJS sandbox is already bypassed regardless of version or context.
- **Source:**
  - Title: DOM based AngularJS sandbox escapes
  - Author: Gareth Heyes (PortSwigger Research)
  - URL: https://portswigger.net/research/dom-based-angularjs-sandbox-escapes
  - Date: 2017-05-11
  - Disclosed report: none
  - Raw file: `sources/xss/15-ps-dom-based-angularjs-sandbox-escapes.md`

Tags: csti, angularjs, sandbox-escape, dom, csp-bypass
