# Bypassing DOMPurify again with mutation XSS (comment/CDATA re-parse)

- **Vuln type / sub-variant:** Mutation XSS research. Technique: defeat DOMPurify's patch for Bentkowski's MathML bypass by hiding the mutation inside a comment (Chrome) or CDATA section (Firefox) rather than a text node.
- **Target type:** Library/app class: anything using DOMPurify (post-2.0.17 patch, up to 2.1) in the `sanitize() -> innerHTML` pattern; also useful as a generic HTML-filter bypass.
- **Recon signal:** Tester recognizes the target is exploitable when sanitized output is re-parsed into the DOM and the sanitizer allows MathML/SVG integration elements plus `style`. The specific tell for this variant: the sanitizer closes the known text-node mutation hole but does not normalize comments / CDATA, so a mutation smuggled through those node types survives.
- **Root cause (mechanism):** Bentkowski's original bypass used `style` with MathML tags to confuse the HTML parser; DOMPurify patched it by checking for potential mutation inside *text nodes*. The gap: the patch did not account for *comment* nodes (Chrome) or *CDATA* sections (Firefox). Inside a MathML/SVG foreign-content context (`math`, `mtext`, `mglyph`, plus `table` and `style`), the parser treats comment/CDATA content differently on re-serialization and re-parse. An encoded close-comment plus an `img`-with-`onerror` placed in a `title` attribute re-materializes as a live element when the browser re-parses the serialized output, because the namespace/insertion-mode boundary shifts between the two parses.
- **Exploitation steps (technique in plain steps):**
  1. Open MathML foreign content and a `style` element so the HTML parser is in a non-HTML insertion mode: `<math><mtext><table><mglyph><style>...`.
  2. Inside `style`, place a comment (Chrome) or `<![CDATA[ ]]>` (Firefox) that the first parse treats as inert.
  3. Encode the comment/CDATA close plus the real payload inside an attribute value (e.g. `title="--&gt;&lt;img src=1 onerror=...&gt;"`), using entities so it survives sanitization.
  4. Firefox variant additionally needs a closing `mglyph`; the author used `&Tab;` entities inside the injected `img` to prove genuine mutation (the entity decodes only on re-parse, not via plain attribute injection).
  5. Sanitize, then let the host assign the result to `innerHTML`; the re-parse turns the hidden markup into an executing `img onerror`.
- **Bypasses used:** Comment-node and CDATA-node smuggling, MathML/SVG namespace confusion, `style` re-parse, entity encoding of the payload (`&gt;`, `&lt;`, `&Tab;`).
- **Impact:** XSS against sites using the then-current DOMPurify. Chrome vector disclosed to Cure53 and patched in DOMPurify 2.1; the underlying browser mutation persisted in both Chrome and Firefox and remains useful for bypassing bespoke HTML filters.
- **Bounty:** n/a (research).
- **Lesson / pattern:** A sanitizer that only inspects text nodes still misses mutations carried by comment and CDATA nodes. mXSS defenses must reason about every node type that the serializer emits and the re-parser can reinterpret, especially across the HTML/MathML/SVG namespace boundary. Treat `math`/`mtext`/`mglyph`/`style` + comment/CDATA combinations as a known mXSS primitive.
- **Source:**
  - Title: Bypassing DOMPurify again with mutation XSS
  - Author: Gareth Heyes (PortSwigger Research)
  - URL: https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss
  - Date: 2020-10-07
  - Disclosed report: none (library issue reported to Cure53, fixed in DOMPurify 2.1)
  - Raw file: `sources/xss/12-ps-bypassing-dompurify-again-mxss.md`

Tags: mutation-xss, sanitizer-bypass, namespace-confusion, dom
