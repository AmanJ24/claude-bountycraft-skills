# XSS without HTML: client-side template injection with AngularJS (CSTI + sandbox escape)

- **Vuln type / sub-variant:** Client-side template injection (CSTI) in AngularJS expressions, chained with an expression-sandbox escape to get XSS. Works even when user input is HTML-encoded and sits inside an attribute, because the attack needs no HTML characters, only `{{ }}`.
- **Target type:** Library/app class: pages served as AngularJS templates (an `ng-app` directive anywhere above the injection point) that reflect user input into the rendered body. Covers Angular 1.x up to 1.5.x where the expression sandbox existed.
- **Recon signal:** Tester recognizes exploitability when `{{7*7}}` (or `{{1+1}}`) reflects back as `49`/`2`, i.e. the page evaluates Angular expressions. The presence of `ng-app` in view-source above the reflection is the key tell. HTML-encoding the output does not help, so a site that "correctly" encodes can still be vulnerable.
- **Root cause (mechanism):** AngularJS renders the page as a template; any reflected `{{expression}}` is evaluated client-side. Angular wrapped expressions in a "sandbox" (`ensureSafeObject`, `ensureSafeMemberName`, `ensureSafeFunction`) meant to block the `Function` constructor, `window`, DOM nodes, `__proto__`, etc. The sandbox is an expression rewriter, not a real security boundary, so version-specific tricks reach the `Function` constructor or inject raw JS into Angular's rewritten output.
- **Exploitation steps (technique in plain steps):**
  1. Confirm expression evaluation with `{{1+1}}`.
  2. Pick a sandbox escape matched to the Angular version (version ranges matter; each branch had its own).
  3. Representative escape families (minimal forms, high level):
     - Oldest (1.0.1 to 1.1.5) and post-sandbox (>=1.6.0, sandbox removed): the `constructor.constructor` approach, i.e. reach the `Function` constructor via an object's `constructor.constructor('alert(1)')()`.
     - Mid versions (Heyes family, ~1.2.24 to 1.4.9): the charAt/charCodeAt override approach, e.g. reassign a native string method such as `'a'.constructor.prototype.charAt=[].join` so Angular's own parser/rewriter mis-tokenizes a crafted `$eval('x=alert(1)//')` and emits attacker JS into the rewritten function body.
     - Others used property-descriptor tricks (`getOwnPropertyDescriptor`/`defineProperty`) or `sort`/`toString` reassignment, again keyed to the version.
  4. The escape causes Angular's rewritten `fn` to contain the attacker's call (the writeup shows the rewrite literally emitting `alert(1)`), which executes as normal JS, outside the sandbox.
- **Bypasses used:** Native-function override (charAt/charCodeAt/fromCharCode re-pointed to `[].join`/`[].concat`) to corrupt both the Angular sanitizer and the expression rewriter; `Function`-constructor access; Unicode-escape and `//` comment tricks to keep the rewritten JS syntactically valid.
- **Impact:** Full XSS on otherwise correctly-encoded Angular sites; the author notes template injection can also appear server-side and lead to RCE.
- **Bounty:** n/a (research). Reported to Google, sandbox patched in 1.5.0, then removed entirely in 1.6.
- **Lesson / pattern:** On an Angular page, treat user-controlled `{{ }}` as code execution: HTML-encoding does not stop CSTI, and the sandbox is not a trust boundary (Angular says so). Confirm with `{{7*7}}`, then apply a version-matched escape. Fix: never reflect user input into Angular templates; do not rely on sandbox/version for safety.
- **Source:**
  - Title: XSS without HTML: Client-Side Template Injection with AngularJS
  - Author: Gareth Heyes (PortSwigger Research)
  - URL: https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs
  - Date: 2016-01-27
  - Disclosed report: none (reported to Google, patched 1.5.0)
  - Raw file: `sources/xss/14-ps-xss-without-html-angularjs-csti.md`

Tags: csti, angularjs, sandbox-escape, reflected, dom
