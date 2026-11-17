---
title: "DOM XSS on http://talks.lystit.com"
author: gamer7112
url: https://hackerone.com/reports/1031644
date_published: 2021-02-09
date_scraped: 2026-11-16
platform: hackerone
disclosed_report: https://hackerone.com/reports/1031644
bounty: awarded (amount hidden/not stated)
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# DOM XSS on http://talks.lystit.com

## Report metadata (HackerOne public JSON)

- Program: lyst
- Reporter: gamer7112
- Submitted: 2020-11-11T15:09:33.403Z
- Disclosed: 2021-02-09T11:38:16.383Z
- State: Closed / Resolved
- Weakness: Cross-site Scripting (XSS) - DOM
- Severity: {"rating": "low", "author_type": "Team"}

## Report body

#Description
DOM XSS can be achieved via a postMessage due to an insecure postMessage handler being registered.

#POC
1. Visit https://gamer7112.com/lyst_1.html
2. Click the link
3. View alert

#Vulnerable Code
Located at http://talks.lystit.com/data-saloon-presentation/plugin/notes/notes.html
```javascript
window.addEventListener('message', function(event) {
    var data = JSON.parse(event.data);

    // No need for updating the notes in case of fragment changes
    if (data.notes !== undefined) {
        if (data.markdown) {
            notes.innerHTML = marked(data.notes);
        } else {
            notes.innerHTML = data.notes;
        }
    }

    silenced = true;

    // Update the note slides
    currentSlide.contentWindow.Reveal.slide(data.indexh, data.indexv, data.indexf);
    nextSlide.contentWindow.Reveal.slide(data.nextindexh, data.nextindexv);

    silenced = false;

}, false);
```

## Impact

XSS allows for an attacker to execute arbitrary javascript on another user.
