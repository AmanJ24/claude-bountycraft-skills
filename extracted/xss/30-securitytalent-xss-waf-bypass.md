# XSS WAF bypass: the ultimate deep dive (SecurityTalent)

Reference / methodology writeup on defeating modern web application firewalls for XSS. Partial: the source file ends mid-sentence inside the closing authorization disclaimer. Captured as technique categories plus case-study summaries, with minimal representative tokens only.

- **Vuln type / sub-variant:** XSS delivered through a payload crafted to evade a WAF. Core thesis: WAF bypass is about parser differentials between the WAF, the browser, and the application framework.
- **Target type:** Apps fronted by ModSecurity/CRS, Cloudflare, Azure Application Gateway WAF, AWS WAF (CloudFront/ALB), or Akamai, where a reflected/stored sink exists but a WAF blocks obvious payloads.
- **Recon signal:** Fingerprint the WAF (response headers/cookies: `cf-ray`, `x-amz-cf-pop`, `x-ms-request-id`, `Server: AkamaiGHost`), then map which characters survive to the response and which simple vectors return 403, revealing the normalization pipeline and parser gap.
- **Root cause:** A WAF has its own parser and normalization pipeline that rarely matches the browser's. Signature matching, incomplete decoding, content-type blind spots, and request-size limits create gaps where a payload the WAF does not recognize still renders as script in the browser.
- **Technique categories (minimal representative token each):**
  1. Context-based breakouts: tailor to HTML-tag, script-string, attribute, or URL/href context. Token: attribute breakout `"><svg ...>`.
  2. Encoding & obfuscation: multi-layer URL encoding, Unicode normalization (fullwidth chars) and JS obfuscation to hide keywords. Token: `%3Csvg...` / `＜svg...＞`.
  3. HTML/CSS injection: CSS expressions (legacy IE), style-tag injection, meta-refresh. Token: `<style>`-based vectors.
  4. Mutation XSS (mXSS): markup that the WAF sees as inert but the browser re-parses into an executable element, for example a `<noscript>`/attribute reparse yielding an `onerror` image. Hard for WAFs to detect because it only becomes dangerous after browser reparsing.
  5. DOM clobbering & prototype pollution: non-script injection that influences DOM/property lookups to reach a sink.
  6. Polyglot payloads: single strings valid across multiple contexts.
  7. WAF-specific bypasses: ModSecurity/CRS (inspect GET vs body, JSON/multipart content-type confusion, request-size exhaustion past the inspection limit, paranoia-level gaps); Cloudflare (dangling markup, JSON endpoint not inspected, Worker); Azure App Gateway (inline comment `/**/` space replacement, tab/newline injection, data URI, event-handler variations); AWS WAF (multipart boundary/case confusion, base64 in query params, null-byte prefix).
- **Case studies (summary level):**
  1. Cloudflare bypass via JSON endpoint: WAF inspected form-encoded bodies on `/search` but not `application/json` to `/api/search`; the query was rendered unsanitized. Stored XSS, $2,500.
  2. Azure WAF bypass via inline comment: CRS 3.2 blocked `<script>` but did not normalize `/**/` before tokenizing, so `<svg/**/onload=...>` passed. Reflected XSS on a login error message, $1,500.
  3. AWS WAF multipart boundary bypass: multipart inspection was case-sensitive on `Content-Disposition` and did not fully parse boundary content when binary data was present, so a caption field carried `<svg onload=...>`. Stored XSS on profile pages, $3,000.
- **Bypasses used:** parser-differential exploitation across encoding, mutation, content-type confusion, request-size exhaustion, uncommon tags/handlers, and WAF-vendor-specific normalization gaps.
- **Impact:** Reflected and stored XSS behind major commercial WAFs; the writeup also covers blind-XSS detection and cookie-theft evasion.
- **Bounty:** case studies list $2,500, $1,500, and $3,000 (other amounts not stated).
- **Lesson / pattern:** WAFs are not a substitute for output encoding and sanitization. The strongest bypasses come from context confusion where the WAF and browser disagree, and from untouched channels (JSON/multipart, GET body, size limits). Normalize consistently, inspect all content types, and fix the underlying XSS.
- **Source:**
  - Title: XSS WAF Bypass: The Ultimate Deep Dive
  - Author: SecurityTalent
  - URL: https://github.com/SecurityTalent/write-up/blob/main/XSS%20WAF%20Bypass.md
  - Date: 2026-06-08
  - Disclosed report: none
  - Raw file: `sources/xss/30-securitytalent-xss-waf-bypass.md`
  - Note: source ends mid-sentence in the closing authorization disclaimer (partial).

Tags: waf-bypass, encoding, polyglot, prototype-pollution, methodology, filter-bypass
