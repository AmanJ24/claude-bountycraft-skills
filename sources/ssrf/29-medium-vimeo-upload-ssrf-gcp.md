---
title: Vimeo upload function SSRF
author: Sayed Abdelhafiz (dPhoeniixx)
url: https://medium.com/@dPhoeniixx/vimeo-upload-function-ssrf-7466d8630437
date_published: 2019-12-13
date_scraped: 2026-11-16
platform: medium
disclosed_report: https://hackerone.com/reports/549882
bounty: not stated (paid in 2 days, amount not given)
vuln_class: ssrf
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->
# Vimeo upload function SSRF

Sayed Abdelhafiz, 4 min read, Dec 13, 2019. Tags: Bughunting, Vimeo, Hackerone, Ssrf.

## TL;DR

I have found an SSRF vulnerability by exploiting content partial flow which the Vimeo upload function implements in the uploading process.

## The Journey

I was studying the most subject I hate and decided to take a rest, I told myself that I wouldn't lose anything If I hunted on my lovely target in HackerOne "Vimeo", I bring my MacBook and I went direct to upload function specifically Google Drive upload feature, I chose to upload from google drive, chose video file, back to BurpSuite to catch the request:

[image: Upload Request]

As you can see, It sends the file URL with google authorization to let the backend server fetch the file from the URL and pass the authorization on the header to access the file from google drive.

let's setup WireShark in our VPS and make the backend request a video file from our server to see what would happen in the backend, that was the result:

[image: HTTP Stream Of Download request]

I noticed unusual headers, like `Range`, `Content-Range`. But its name was enough to tell me what it's work. we can assume that most of the time, the size of video files is big, so Vimeo doesn't request the full file in one request if the size was big. if the video file is small, It will request the full file, if not It will request the file partially until collecting the full file, the following diagram can describe what happens:

[image: Normal person uploading]

The server sends a request to check the file size, then if the file was small as it can be transferred to the server by a single connection, it will request the full file.

**Suddenly**, an idea came to me, what if I didn't send the full file to the server? For example, if my server responded to the backend server telling it that the file length was `500B`, the server will request the `500B`, now what if I responded to the server with `200B`of data? let's try it ;)

I have coded a Web Server in Python to test it, The full file length is `554231B`, when Vimeo requests the full file, my server will respond by `8228B` Only! my server will log Vimeo `Range` the header for me :)

[image: script output]

Cool! the server stored the `8228B`, requested the rest of the file, Cool! I have a scenario that can lead to SSRF in this case.

What If my server responded with a redirect response? vimeo will follow the redirect? store the response? request me the rest? _If there rest_

[image: diagram]

some web servers restricted me to reproduce the attack, So I decided to code a pure HTTP server to perform this attack, I coded, ran it, and passed my attacker host to the upload function, **SURPRISE!** my attack works fine! my thoughts were right D:

_Notice: in Vimeo you can download your original video file, so you can find the request forgery response easily_

## Time to exploit

I have gathered information about the server, it was a google cloud Instance, so lets try to retrieve instance metadata, I changed the redirect target to `http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token`, reproduced the attack and I was able to get their compute engine API access token! it was a great moment for me because it was the first SSRF with the most critical impact I have ever found.

## What now?

Sent to Vimeo, resolved, and paid in 2d!

## Timeline

Apr 29th: Triaged by HackerOne team.

Apr 29th: Vimeo rewarded me initially.

May 1st: The vulnerability has been fixed.

May 1st: Vimeo rewarded me with the rest of the bounty.
