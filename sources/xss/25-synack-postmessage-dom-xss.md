---
title: Exploiting PostMessage Handlers to Achieve DOM XSS
author: John Kounelis (Synack Red Team)
url: https://www.synack.com/exploits-explained/exploiting-postmessage-handlers-dom-xss/
date_published: 2026 (page modified 2026-08-20; exact publish date not shown)
date_scraped: 2026-11-16
platform: personal-blog
disclosed_report: none
bounty: not stated
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

<!-- SCRAPE NOTE: the code listings/PoCs in this post are embedded as images (Sending-data_1, Receiving-Messages_1/2, Example-1_1/1_2, Example-2_1/2_2, Example-3_1/3_2) and were NOT extractable as text. Only the prose and inline snippets below survive. -->

# Exploiting PostMessage Handlers to Achieve DOM XSS

The postMessage API was designed to enable safe cross-origin communication in the browser. In practice, it is often implemented in a way that can allow malicious sites to exchange data with a web page. Missing or improper origin validation opens the door for attackers, and when untrusted message data flows into a DOM sink, the result is DOM XSS.

In this post, I walk through examples of vulnerable postMessage handlers that were uncovered during various Synack Red Team engagements.

## How postMessage Handlers Work

The Same-Origin Policy is a browser-enforced security mechanism that prevents websites on different origins from interacting with one another. However, there are scenarios in which two cross-origin pages would like to be able to communicate with one another on the client side. This is where postMessage comes into play, as it provides a way to have pages send and receive data.

### Sending data

From a web page, a message can be sent to another window reference (e.g. an iframe or popup window) using the postMessage() function.

[image: code listing, not extracted]

### Receiving Messages

Of course, sending data via postMessage is only meaningful if the other page is willing to receive messages. This can be done by creating an event listener, which will trigger a function call when a certain event happens (in this case, when a message is received).

[image: code listing, not extracted]

When a message is received, the page will receive not just the data being sent, but a MessageEvent which will automatically be populated with certain properties such as origin. The origin property represents the origin of the sender page and is often used to verify if the message came from a trusted site.

[image: code listing, not extracted]

In practice, however, these origin checks are often missing or are not implemented securely, allowing malicious web pages to communicate through the listener. While this may not be inherently dangerous, if the listener is using data from the message to perform actions like manipulate the DOM, it can lead to DOM-based XSS.

## Example 1: No Origin Check

While hunting on a target, I came across a blank page https://site.vulnerable.com/page that looked completely uninteresting on the surface. However, popping open devtools revealed there was more than meets the eye.

[image: devtools screenshot, not extracted]

There was an iframe on the page as well as a postMessage listener with no origin check. The listener would use the data received to redirect the iframe to an arbitrary URL. This meant a malicious page could send a message to redirect the iframe to javascript: URI and achieve XSS on the parent page.

The only caveat was that the iframe was on a different origin, so sending a message containing {"URL":"javascript:alert(document.domain)"} would give an error:

_Unsafe attempt to initiate navigation for frame with origin 'https://other.site.com' from frame with URL 'https://site.vulnerable.com/'. The frame attempting navigation must be same-origin with the target if navigating to a javascript: url_

The same origin policy blocks javascript: URI navigations on cross-origin iframes. However, this wasn't a problem because we could simply redirect the iframe to a same-origin page, then redirect it once again to a javascript: URI to get XSS. The proof-of-concept looked as follows:

[image: PoC code, not extracted]

The malicious page would open the vulnerable page and send data via postMessage to redirect the iframe to https://site.vulnerable.com/ after 3 seconds to make the iframe src same-origin. Then, after a few more seconds, the javascript: URI payload would be sent and pop XSS.

## Example 2: Origin Check Bypass

Another scenario involved a postMessage listener on a target page that looked similar to the following:

[image: listener code, not extracted]

At first glance, this may appear to be performing an origin check, but there was a crucial mistake. event.origin.includes("targetsite.com") is only checking that the sender's origin contains "targetsite.com". This can be bypassed by creating a subdomain such as https://targetsite.com.attacker.com. From there, the handler was passing input into a script src, which can be used to load javascript into the page from a URL. XSS could be achieved with the following PoC:

[image: PoC code, not extracted]

## Example 3: Null-Origin Bypass

In this scenario the target had a listener that was similar to the following:

[image: listener code, not extracted]

### How to Secure postMessage Listeners

originCheck() returns true if origin === window.origin. The intention appears to be to permit messages sent from a same-origin page, but this can actually be bypassed by using sandboxed iframes, which will set the MessageEvent's origin as well as window.origin of any window opened within the iframe to "null." So, e.origin and window.origin would both be set to "null" and allow the message through. Then, the handler would write input directly to the DOM with innerHTML in setContent(). The PoC below illustrates the exploit:

[image: PoC code, not extracted]

originCheck() should only allow origin === window.location.origin to validate same-origin messages, as window.origin is prone to manipulation via sandboxing.

## Frequently Asked Questions

**If the Same-Origin Policy blocks javascript: URI navigation on a cross-origin iframe, how is DOM XSS still possible through a postMessage handler that redirects that iframe?**

The block only applies while the iframe is cross-origin. If the handler accepts an arbitrary URL, an attacker can first send a message that redirects the iframe to any same-origin page on the vulnerable site. Once the iframe is same-origin with the parent, a second postMessage carrying a javascript: URI will execute. This two-step redirect is exactly the pattern shown in Example 1.

**What makes a substring-based origin check like event.origin.includes("targetsite.com") insecure?**

includes() returns true whenever the trusted string appears anywhere in the sender's origin, not just as the registrable domain. An attacker only needs to stand up a host like https://targetsite.com.attacker.com, and the check passes. As shown in Example 2, this opens the door for any message payload the handler trusts, including one that injects a script tag pointing at attacker-controlled JavaScript. Origin validation should use strict equality against an allowlist of trusted origins.

**Why is comparing event.origin to window.origin not a reliable way to allow only same-origin messages?**

window.origin is not trustworthy when the page can be loaded from a sandboxed iframe. A sandboxed iframe sets its MessageEvent origin and the window.origin of any popup it opens to the literal string "null", so both sides of the comparison match and the message is accepted. As the closing note in Example 3 calls out, same-origin checks should use window.location.origin, which reflects the page's actual URL and cannot be forced to "null" through sandboxing.

## About the Author

John Kounelis is a Senior Product Security Engineer with a strong foundation in software engineering and application security. With over five years of experience, he specializes in web application pentesting, source code review, and secure software development practices. He is an active member of the Synack Red Team.
