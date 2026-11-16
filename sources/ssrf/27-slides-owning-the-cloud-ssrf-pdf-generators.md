---
title: "Owning the Cloud Through SSRF and PDF Generators (AppSec California 2020 slides)"
author: Ben Sadeghipour (@NahamSec) and Chris Holt (@humor4fun), with Cody Brocious
url: https://static.sched.com/hosted_files/appseccalifornia2020/11/Ben%20Sadeghipour%20-%20Owning%20the%20clout%20through%20SSRF%20and%20PDF%20generators%20-%20ORIGINAL.pdf
date_published: 2020 (AppSec California 2020)
date_scraped: 2026-11-16
platform: other
disclosed_report: none
bounty: "not stated"
vuln_class: ssrf
---

<!-- Raw scraped text below, from a 78-page slide PDF parsed to text. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim.
     Cleaning: the parser repeated each bullet several times and emitted "[Image: ImageNN]" placeholders; duplicates and placeholders removed, slide order kept, Verizon legal safe-harbor boilerplate omitted. Payloads shown on image-only slides (ERB injection to RCE, screenshots) are not recoverable from text. -->

# OWNING THE CLOUD THROUGH SSRF AND PDF GENERATORS

Ben Sadeghipour, Chris Holt. AppSec Cali 2020

## About me

Ben Sadeghipour
- Also known as @NahamSec
- Head of Hacker Education at HackerOne
- Top 20 hacker on HackerOne
- Most P1/P2's on Bugcrowd in July
- Snapchat, VerizonMedia, Dept. of Defense, Airbnb, Valve, Rockstar Games, etc.
- Makes hacking videos on YouTube, streams hacking on Twitch every Sunday/Tuesday

Also a big thank you to Cody Brocious for helping with a lot of this research.

Chris Holt
- Also known as @humor4fun, commonly known as flyingtoasters
- Senior Bug Bounty Operations Lead, Verizon Media
- Certified by GAIC, PADI, and formerly USSF
- Datacenter enthusiast, homelabbing since 2004

Shoutout to the rest of The Paranoids and the product teams at Verizon Media.

## SSRF According to OWASP

In a Server-Side Request Forgery (SSRF) attack, the attacker can abuse functionality on the server to read or update internal resources. The attacker can supply or a modify a URL which the code running on the server will read or submit data to, and by carefully selecting the URLs, the attacker may be able to read server configuration such as AWS metadata, connect to internal services like http enabled databases or perform post requests towards internal services which are not intended to be exposed.

TL;DR: Make requests using the target host and in some cases render JS server side

## What is Cloud Metadata?

- 169.254.169.254 - accessible internally within the machine you have access to.
- Provides details like internal IP, hostname, project details, etc.
- And if you're lucky enough, it could also give you access to access_key & secret_key as well

## Basic Example

- Upload avatar via URL and triggers the following request:

```
GET /api/v1/fetch?url=https://site.com/myfunnycatmeme.jpeg
Host: thesiteweareabouttpwn.com
```

- Changing the URL parameter to something.internal.target.com may give us access to see internal assets
- Not limited to http, you can use other protocols
  - file:///etc/passwd
  - gopher://
  - ssh://

## CVE Examples

- JIRA CVE-2017-9506: pointing consumerUri to Google; then to metadata. Reference: https://medium.com/bugbountywriteup/piercing-the-veil-server-side-request-forgery-to-niprnet-access-c358fd5e249a
- Jenkins CVE-2018-1000600: pointing apiUri to AWS Metadata

## SSRF Hurdles

Sometimes it's not as straightforward as a single http request. In some cases you may be dealing with filters or you may not even see the output of your request but you still have a few options.

- Problem: metadata or internal IPs are getting filtered
  - Solution: Use a custom domain like meta.mydomain.com and point it to the asset you are trying to access (aws.mydomain.com -> 169.254.169.254)
- Problem: Only able to use whitelisted domains
  - Solution: Find an 'Open Redirect' on the whitelisted domain(s) and use that to exploit your SSRF
- Problem: SSRF is there but I can't see the output
  - Solution: Use Javascript and exfil data

## Valuable Assets / Vulnerabilities

- Knowing the User Agent (more on this later)
- XSS on the target application where it also gets pushed to the PDF
  - How to confirm it: `<script>document.write(123)</script>`
  - Generate PDF and it should print 123
- Follows redirection by pointing the url or HTML tag (iframe/img etc) to our host where redirect.php redirects to success.php
  - mysite.com/redirect.php -> redirects to mysite.com/success.php
- Any customization that involves HTML/CSS (Font name, colors, styling)
- Open redirect on the target application in case of any domain whitelisting

## PDF generation process

### Headless Browsers

There are two common headless browsers in use:
- wkhtmlpdf is a Webkit implementation whose rendering backend is PDF.
- Headless Chrome is desktop Chrome minus the GUI and with a PDF or image rendering backend attached.

Lots of wrapper libraries providing easy integration with any language.

### HTML Renderers

Rather than using an actual browser engine, these renderers work by doing the HTML and CSS parsing, without any kind of JavaScript support or dynamic layout engine.
- tend to provide a restricted environment where most HTML can be handled safely and efficiently
- WeasyPrint is a great example of this class of HTML->PDF converters (more later)

## XSS in PDF files: PDF + XSS == SSRF

Most modern web applications performing PDF generation do not actually generate PDFs directly.
- As such, any XSS into this data gets you running in the context of the server, not the client!

The attack strategy used will depend on what conversion system is in use in the application, but these can be broken into two categories: Headless browsers and HTML renderers.

## Simple XSS->SSRF via wkhtmltopdf

You notice HTML is rendering within your Generated PDF
- we want to make sure this can communicate with other hosts
- `<iframe src="http://myhost:myport:443">` (or anything that'll fetch something from a remote host)
- This also gives us the 'User Agent'

Then:

```
<iframe src="http://169.254.169.254/user-data/[...]">
```

## HTML renders but...

- Most user input gets sanitized/filtered
- We haven't found an XSS in our target app
  - But... we are allowed to customize the fonts and styling of the generated PDF

## XSS by escaping </style> tag

- Most user input gets sanitized/filtered; no XSS, but we are allowed to customize the fonts and styling
- Confirm it renders HTML within the PDF generator
- Can it fetch anything from a remote host? It hits your server once you download your PDF.

Replace test payload with:

```
</style><iframe src="http://169.254.169.254/user-data/[...]">
```

and extract data.

No iframe? No problem! Use an open redirect:

```
</style><iframe src="https://mysite.com/redir.php?url=http://169.254.269.254/[...]">
```

No iframe? Only whitelisted domains? No problem. Find an open redirect:

```
</style><iframe src="https://target.com/something/redirect?url=http://169.254.269.254/[...]">
```

(Note: slide text reads 169.254.269.254 as printed; the metadata address is 169.254.169.254.)

## But it's not always so (W)easy: WeasyPrint

Once you know the trick, at least.

This one stumped us for a while. We got HTML Injection into a PDF no problem, but there were two things that made this hard:
1. It didn't seem to run any scripts, load iframes, or seemingly do anything but load images.
2. Every single payload we wanted to test required us to take a rideshare somewhere.

### Use the source

Once we got it to connect to a server where we could see the request, we noticed that the user agent said it was from WeasyPrint. A quick Google search later and we learned it was a pretty straightforward HTML renderer written in Python and it was open source!

Thankfully, we could run this locally and render pages just like the victim.

Unfortunately, this was when we got really pessimistic. This thing didn't render anything fun. Text, some CSS, images -- that was about it.

- How does it work? `weasyprint input.html output.pdf`
- Only fetched images
- No Javascript
- No `<iframe>`
- Html.py from WeasyPrint's GitHub repository indicates the supported tags: `<img>` (dead end), `<embed>` (dead end), `<object>` (dead end), `<link>` (interesting)

### Attachments

```
<link rel=attachment href="file:///etc/passwd">
```

This embeds files right into the PDF itself! They aren't visible on the page, but they're included as a hidden resource on the file. We could not only read files, but make web requests. Three rideshares later, we had their full EC2 access keys. (We originally tried this locally with a large file to see if it'll increase our PDFs file size... and it did.)

Slide: "Unpacks the content from pdf".

## DNS Rebinding for Fun and Profit

DNS rebinding attacks provide a means to get around this. We make the browser think it's requesting data from the same domain the page was loaded from and it's game over.

1. Browser loads http://ex.ploit.info/ and the script sends a message to the server to rebind ex.ploit.info to 169.254.169.254
2. The script then resolves a0.ex.ploit.info through a2499.ex.ploit.info, flushing the DNS cache for the original domain
3. Then the script can request any data from the metadata service using requests to ex.ploit.info; the metadata services don't care what hostname is used to make requests to them
4. Data can be sent to bc.ex.ploit.info, which serves as a backchannel for exfiltration

(Slide: "Start editing your Snap by selecting a layer from the list on the left side of the screen." appears as a screenshot of a Snap-related finding; details not in text.)

## HTML Injection -> SSRF -> XXE in PDF generators

HTML works but no XSS:
- Iframe tricks don't work: no redirect, no local file read, no XSS. No problem.
- `<IMG>` works and it leaked the user agent as Prince v10 (CVE-2018-19858)
  - Prince converts "HTML, XHTML, or one of the many XML-based document formats"
  - PRINCEXML UP TO 10 XML DATA XML EXTERNAL ENTITY

```
<?xml version="1.0"?> <!DOCTYPE cdl [<!ENTITY asd SYSTEM "http://169.254.169.254/[...]">]> <cdl>&asd;</cdl>
```

Reference: https://www.corben.io/XSS-to-XXE-in-Prince/

## BONUS: ERB injection in PDFs

- Template injections have become common for both client side (XSS) and server side (SSRF, local file read, RCE in some cases)
- Not hard to identify, just add them to your payloads
  - For example: `<h1>test123<%= 7 * 7 %> {{7*7}}`
  - Returns `test123 49 {{7*7}}`
- Sometimes you may not be expecting it, especially when it comes down to PDF generators
- Originally identified as a potential SSRF due to HTML Injection. But we had no iframe, no XSS, no way to redirect, etc.
- No problem.. We have the User Agent
- Collaboration is key!
- Originally thought about throwing Java at it.. No luck
- But throwing `<%= 7 * 7 %>` returned 49 so there's hope!
- So let's see if we can get a `self.class.name` to return
- Escalate to RCE (payload on image-only slides)

## SSRF tools

### HTTPRebind

Rebinding attacks can be very valuable for SSRF, but they require a lot of setup work, tweaking, and programming. HTTPRebind combines a DNS server with an HTTP server to automatically handle all of this for you.
- Usable against any headless browser
- Takes only seconds to run due to DNS cache flushing
- Automatically pulls critical data from GCP, AWS, and Azure

Source: https://github.com/daeken/httprebind

### SSRFTest

This tool lets you quickly do a first-pass test for SSRF. It will record incoming requests for your different targets as well as automatically attempt to access and dump data from EC2 metadata service.

The optimal targets for SSRFTest's automated functionality are real headless browsers living in the cloud, but it's a useful starting point for any SSRF exploitation.

Code: https://github.com/daeken/SSRFTest ; public instance: https://ssrftest.com/

## SSRF, Bug Bounty & Verizon Media (Chris Holt section)

"We are the Paranoids. We protect consumer, customer, and corporate data."

### Classifying SSRF bugs

- Open Redirect (301 Moved Permanently, 302 Temporarily Moved)
- Blind SSRF (401 Not Authorized, 404 Not Found)
- Content-Based SSRF
- Full SSRF

### "There's always money in the Banana Stand" (Verizon Media SSRF testing/triage setup)

- Build it: internal canary hosts such as banana.stand.ne1.yahoo.com and Banana.stand.cgq1.yahoo.com serving test files (json_001.json, png_001.png, and many other extensions: avi, bmp, css, csv, doc, docx, dtd, flv, gif, html, icns, ics, ico, jar, jpg, js, json, md, mkv, mov, mp3, mp4, odp, ods, odt, ogg, pdf, php, png, ppt, rss, svg, tiff, txt, wav, wmv, xls, xlsx, xml, xsl, zip). "In is not always in."
- Log it: logging, better logging, log alerts (Corp "We are watching", Production, Internet diagram)
- Prove it
- The one and only

## Recap

- SSRFs can be very dangerous
- Don't give up on your bugs until you have tried every possible scenario
  - WeasyPrint took us ~3 months to piece together
- If you see a PDF generator somewhere, 9/10 it's vulnerable
  - Especially if you chain with other vulnerabilities (XSS, Open Redirect, etc)

Defender recap:
- Disable Javascript
- Create some good whitelisting
- Properly configure your cloud instances to minimize impact
- Be nice to hackers

Thank you! Special thanks to: Orange, Alyssa Herrera, Shubs (infosec_au), Ziot, Daeken, cdl
