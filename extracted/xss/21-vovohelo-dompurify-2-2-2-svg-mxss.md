# vovohelo: From SVG and back, DOMPurify <2.2.2 mXSS bypass

- **Vuln type / sub-variant:** Mutation XSS via namespace confusion (HTML to MathML to HTML to SVG collapsing to HTML to MathML), variant of the <2.0.17 bypass. Raw file is full and not paywalled; two embedded gists and all figures were not captured.
- **Target type:** Apps using DOMPurify <2.2.2 with the usual `innerHTML = DOMPurify.sanitize(...)` pattern.
- **Recon signal:** Building on Bentkowski's and Gareth Heyes's mglyph bypasses: the author asked what happens if SVG is mixed into the chain, and used the LiveDOM++ tool to inspect trees.
- **Root cause:** Same double-parse issue. An "ownership mutation gadget" (nested `form`, also `table`) changes the direct parent of a homograph tag (`mglyph`, `style`) on re-parse, which changes its namespace. Here the tag chain is harmless in the SVG namespace but dangerous in MathML.
- **Exploitation steps:**
  1. Final payload: `<form><math><mtext></form><form><mglyph><svg><mtext><style><path id="</style><imgonerror=alert(1)src>">`
  2. Sanitizer view: inside `svg`, `mtext` is just an unknown element, `<style>` children are parsed as elements, and `</style><img...>` sits inside the SVG `path` `id` attribute value, a safe free-text attribute.
  3. After serialization and re-parse the nested `form` is dropped, so `mglyph` (with its `svg` subtree) lands under the `mtext` text integration point and ends up in MathML. Then `mtext` is a MathML text integration point, so its children are in the HTML namespace, `<style>` becomes raw text until `</style>`, and the formerly attribute-bound `<img onerror>` is created as real markup.
- **Bypasses used:** Nested-form ownership gadget; namespace chain via foreign content; payload hidden in an SVG `path id="..."` attribute; the tag written as `<imgonerror=alert(1)src>` (no spaces) in the attribute-side text.
- **Impact:** Full DOMPurify bypass in <2.2.2 with default config. Reported 2020-11-02 and fixed in 2.2.2 the same day.
- **Bounty:** n/a (not stated)
- **Lesson / pattern:** After one mXSS fix, look for the same ownership gadget with a different tag chain; hide the real payload in an attribute or text that is inert in one namespace and markup in another.
- **Source:**
  - Title: From SVG and back, yet another mutation XSS via namespace confusion for DOMPurify <2.2.2 bypass
  - Author: Daniel Santos (vovohelo / bananabr)
  - URL: https://vovohelo.medium.com/from-svg-and-back-yet-another-mutation-xss-via-namespace-confusion-for-dompurify-2-2-2-bypass-5d9ae8b1878f
  - Date: 2020-11-11
  - Disclosed report: none
  - Raw file: `sources/xss/21-vovohelo-dompurify-2-2-2-svg-mxss.md`

Tags: mutation-xss, sanitizer-bypass, namespace-confusion, svg, gadget
