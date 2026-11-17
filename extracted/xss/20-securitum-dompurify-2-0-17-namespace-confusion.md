# Bentkowski: mXSS via namespace confusion, DOMPurify <2.0.17 bypass

- **Vuln type / sub-variant:** Mutation XSS (sanitizer bypass) using nested-form mutation plus MathML/HTML namespace confusion. Raw file is full text; DOM-tree screenshots and one follow-up payload (image only) are not captured. Note: the raw file's header URL differs slightly from the candidate list URL (same article).
- **Target type:** Apps using `div.innerHTML = DOMPurify.sanitize(html)` (DOMPurify <2.0.17).
- **Recon signal:** The standard usage pattern parses the markup twice (sanitize, serialize, `innerHTML` re-parse). Any spec-compliant mutation in that round trip is a candidate bypass.
- **Root cause:** Parse, serialize, parse again is not guaranteed to reproduce the same DOM tree. The HTML spec itself shows nested `<form>` can exist after the first parse but the inner form is dropped on re-parse, which moves children to a different parent and so to a different namespace.
- **Exploitation steps:**
  1. Final payload: `<form><math><mtext></form><form><mglyph><style></math><img src onerror=alert(1)>`
  2. First parse (what DOMPurify sees): nested `form`s exist, `mglyph` is in the HTML namespace, so `<style>` is plain text and the `<img onerror>` is just text inside it. All elements are allow-listed, so nothing is removed.
  3. DOMPurify serializes: `<form><math><mtext><form><mglyph><style></math><img src onerror=alert(1)></style></mglyph></form></mtext></math></form>`
  4. On `innerHTML` re-parse the second `<form>` is ignored, so `mglyph` becomes a direct child of `mtext` (a MathML text integration point). `mglyph` is a special exception that stays in the MathML namespace, so `<style>` is parsed as an element (not raw text), `</math>` closes, and `<img>` is created in the HTML namespace with a live `onerror`.
- **Bypasses used:** Nested `form` mutation (spec example `<form id="outer"><div></form><form id="inner"><input>`); `mglyph`/`malignmark` namespace exception under MathML text integration points; `<style>` behaving differently in HTML (text only) vs foreign content (child elements parsed). Integration points: MathML text (`mi mo mn ms mtext`) and HTML (`annotation-xml` with `encoding=text/html|application/xhtml+xml`, `foreignObject`, `desc`, `title`).
- **Impact:** Full DOMPurify bypass with default config, leading to XSS wherever sanitized output is assigned to `innerHTML`.
- **Bounty:** n/a (not stated)
- **Lesson / pattern:** Sanitize-then-`innerHTML` is prone to mXSS by design; look for tree-shape changes on re-parse that flip an element's namespace. Mitigation advised: use `RETURN_DOM` / `RETURN_DOM_FRAGMENT` to avoid the serialize/re-parse round trip.
- **Source:**
  - Title: Mutation XSS via namespace confusion, DOMPurify <2.0.17 bypass
  - Author: Michal Bentkowski (Securitum)
  - URL: https://securitum.com/mutation-xss-via-namespace-confusion-dompurify-2-0-17-bypass.html (raw file lists https://securitum.com/mutation-xss-via-mathml-mutation-dompurify-2-0-17-bypass.html)
  - Date: 2020-09-21
  - Disclosed report: none
  - Raw file: `sources/xss/20-securitum-dompurify-2-0-17-namespace-confusion.md`

Tags: mutation-xss, sanitizer-bypass, namespace-confusion, svg, gadget
