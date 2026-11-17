# DOMPurify SVG mutation XSS via namespace confusion (H1 #1024734)

- **Vuln type / sub-variant:** Mutation XSS (mXSS) bypass of DOMPurify's sanitizer, driven by HTML/SVG/MathML namespace confusion. Same technique family as Michal Bentkowski's MathML 2.0.17 bypass.
- **Target type:** Any web app that sanitizes untrusted HTML with DOMPurify (default config, version up to 2.2.0) and then writes the sanitized string back into the DOM via `innerHTML`.
- **Recon signal:** App takes attacker HTML, runs it through DOMPurify, and reinserts the result into a live DOM node. Tester suspects mXSS when sanitized output is re-parsed by the browser (the classic `sanitize() -> innerHTML` round trip) rather than inserted as text.
- **Root cause:** DOMPurify parses the input, decides it is clean, and serializes it. The bug: the DOM subtree the browser builds when that *serialized* string is re-parsed differs from the tree DOMPurify inspected. Foreign-content elements (MathML `math`/`mtext`/`mglyph`, SVG `svg`, and HTML integration points) change the parser's insertion mode, so content that looked inert as a text/attribute node during sanitization gets re-interpreted as live HTML on the second parse. In particular a `style` element lets an attribute value or text that contains an encoded `</style><img onerror=...>` escape its original context when the markup crosses the MathML/SVG/HTML namespace boundary, re-materializing as an executable `img onerror`.
- **Exploitation steps:**
  1. Build markup that opens MathML/SVG integration-point elements (`math`, `mtext`, `mglyph`, `svg`) and a `style` block, then stashes an encoded `img`-with-`onerror` payload inside an attribute or text position that is harmless at sanitize time.
  2. Minimal illustrative shape (not the full weaponized string): `<math><mtext>...<mglyph><svg><mtext><style><path id="</style><img onerror=... src>">`.
  3. Feed it to `DOMPurify.sanitize()`; the output passes the allowlist.
  4. Host page assigns the sanitized output to `element.innerHTML`; the browser re-parses, the namespace boundary shifts, and the hidden `img onerror` becomes a real element that fires.
- **Bypasses used:** Namespace confusion (MathML/SVG integration points), `style` element re-parse, entity-encoding of the inner closing tag so it survives the first parse. No filter list evasion in the classic sense; the allowlist is technically honored but the post-serialization tree differs.
- **Impact:** XSS on any site using DOMPurify default config at the affected version. Reporter notes default-config sites are vulnerable, i.e. a very wide blast radius for a library-level bug.
- **Bounty:** Not eligible. Internet Bug Bounty declined (scope is core internet infrastructure, not a single vendor's product). Fixed by Cure53 in DOMPurify 2.2.2.
- **Lesson / pattern:** Sanitizing a string and then re-parsing it with `innerHTML` is unsafe whenever foreign-content (SVG/MathML) elements can shift the parser's insertion mode. A sanitizer that validates one parse tree cannot vouch for a second, different parse. Defense: keep the sanitizer current, prefer setting output as text or via a parser that does not re-serialize, and treat `style`/`math`/`svg`/`mglyph`/`mtext` combinations as high-risk.
- **Source:**
  - Title: DOMPurify bypass (SVG mutation XSS, Report #1024734)
  - Author: vovohelo (Daniel Santos / bananabr)
  - URL: https://hackerone.com/reports/1024734
  - Date: 2020-11-02 (disclosed 2020-12-17)
  - Disclosed report: https://hackerone.com/reports/1024734
  - Raw file: `sources/xss/09-h1-dompurify-svg-mxss-1024734.md`
  - Note: same researcher as candidate #21 (author's own Medium writeup); treat as complementary.

Tags: mutation-xss, sanitizer-bypass, namespace-confusion, stored, dom
