---
title: From SVG and back, yet another mutation XSS via namespace confusion for DOMPurify < 2.2.2 bypass
author: Daniel Santos (vovohelo / bananabr)
url: https://vovohelo.medium.com/from-svg-and-back-yet-another-mutation-xss-via-namespace-confusion-for-dompurify-2-2-2-bypass-5d9ae8b1878f
date_published: 2020-11-11
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: xss
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim.
     Paywall check: page served in full to an anonymous request, no members-only marker, no truncation.
     Note: embedded GitHub gist snippets other than the final payload and nested-form gadget were not captured (DOMPurify usage sample, "bypass fragment" snippet), and figures are images. -->

# From SVG and back, yet another mutation XSS via namespace confusion for DOMPurify < 2.2.2 bypass

Daniel Santos, Nov 11, 2020, 8 min read

For those who are only interested in the final payload here you go (I won't judge). For the ones interested in why it works, please bear with me.

Snippet 1: final payload (gist mXSS.html, https://gist.github.com/bananabr/bf720f15d8f726f649c464345eedbd40)

```
<form><math><mtext></form><form><mglyph><svg><mtext><style><path id="</style><imgonerror=alert(1)src>">
```

First of all, I would like to point out that this article and the bypass described here are heavily based on Michal Bentkowski (@SecurityMB) research. The original article and previous bypass who made it possible for me to find this new vector are available here: https://research.securitum.com/mutation-xss-via-mathml-mutation-dompurify-2-0-17-bypass/. The article also contains all of the required foundations the reader may need to properly understand how and why the bypass described here works. It took me a few rounds of reading and playing with the LiveDOM++ (https://livedom.lab.xss.academy/) tool, also provided by @SecurityMB, to really understand why Michal's original bypass worked in the first place. Gareth Heyes (@garethheyes) also built upon Michal's work and found a variation of the original bypass days after it got published. I also took my time with Gareth's bypass to properly understand what was going on.

## What is DOMPurify?

DOMPurify (https://github.com/cure53/DOMPurify) is a widely used HTML sanitizer library. It is mainly used to sanitize user input on web applications that permits the creation of HTML/Rich Text content. Think of a web-mail client or blog platform for example. A common usage pattern for DOMPurify is the following (code sample is an embedded gist in the original, not captured).

According to Michal's article:

> In terms of parsing and serializing HTML as well as operations on the DOM tree, the following operations happen in the short snippet above:
>
> `html` is parsed into the DOM Tree;
>
> DOMPurify sanitizes the DOM Tree (in a nutshell, the process is about walking through all elements and attributes in the DOM tree, and deleting all nodes that are not in the allow-list);
>
> The DOM tree is serialized back into the HTML markup;
>
> After assignment to `innerHTML`, the browser parses the HTML markup again;
>
> The parsed DOM tree is appended into the DOM tree of the document;

The important takeaway is that the HTML markup is parsed twice and serialized into a string in between.

## Namespaces, and why they are important

HTML is a markup language based on XML. XML utilizes the concept of namespaces. An HTML document, compliant with the current specification of HTML 5, may contain elements from three different namespaces.

- HTML (`http://www.w3.org/1999/xhtml`)
- SVG (`http://www.w3.org/2000/svg`)
- MathML (`http://www.w3.org/1998/Math/MathML`)

Namespaces solve the ambiguity problem when a single XML document contains homograph elements and/or properties from different "vocabularies". For example, all the previously listed specifications contain a tag named _style._ The _style_ tag has different properties and rendering behaviors associated with it depending on the namespace it's used. This means that a browser will behave differently when it parses a _style_ tag depending on its ancestors (Figure 1: style tag in the HTML namespace, Figure 2: style tag in the SVG namespace).

## DOM mutation in a nutshell

Although it may sound counter-intuitive, parsing and serializing a DOM fragment is not an idempotent operation. This means that in some cases, depending on how the fragment is constructed, the serialized version of a DOM tree won't result in the same DOM tree when parsed. This double-parsing behavior is inherent to DOMPurify's standard usage. However, some unexpected results from double-parsing DOM fragments are not unexpected at all, as they are documented in HTML's current specification (https://html.spec.whatwg.org/). One of these cases (https://html.spec.whatwg.org/multipage/parsing.html#serialising-html-fragments) regards nested forms. A DOM fragment with nested _form_ tags is not to be considered a valid construction according to HTML's current specification. However, the following HTML fragment when parsed once will result in a DOM tree with nested forms.

Snippet 2: nested form mutation gadget (gist nfgadget.html)

```
<form id="outer">
<div>
</form>
<form id="inner">
<input>
```

We can use the LiveDOM++ tool to inspect how the DOM behaves, before and after being sanitized with DOMPurify, when it is fed with the HTML fragment mentioned above.

Figure 3: nested forms parsed twice.

As demonstrated, when parsed for the first time, the HTML fragment results in a non-compliant DOM tree containing nested _form_ tags. After sanitizing it, DOMPurify will serialize the DOM tree and the resulting string will be parsed again by the browser. It is then possible to verify that the direct parenthood of the _input_ tag is transferred the inner _form tag_ to the outer _form_ tag in the final fragment.

We will refer to the type of HTML markup that results in a mutation that changes the direct parent of a tag as an **_ownership mutation gadget_**.

The _table_ tag can also be used to construct an **_ownership mutation gadget_** as shown below (Figure 4: table ownership mutation gadget, image).

In the example above the direct parent of the inner _anchor_ tag is changed from the outer _anchor_ to the _div_ tag. Understanding how these gadgets behave is crucial to understanding the bypass construction. I recommend experimenting with the gadgets on LiveDOM++ to get a better understanding of how the gadgets behave.

## Foreign content

By default, all elements in an HTML document must be parsed by a browser according to the rules defined by the HTML namespace; however, if the parser encounters a `<svg>` or `<math>` tag, it should then parse those elements and their descendants according to the SVG and MathML namespaces respectively. One also needs to consider that both the MathML and SVG namespaces support foreign content as well, meaning a chain of namespaces transitions can be built (Figure 5: html->svg->math->html transition chain).

## Namespace confusion, putting it all together

As mentioned before, homograph elements like the _style_ tag have different properties and rendering rules depending on the namespace they are in. This means that if we use an **_ownership mutation gadget_** to change the direct parent of a homograph tag and cause its namespace to change, we can trick the sanitizer into producing a malicious serialized HTML fragment. That is exactly what Michal did. Michal's original bypass used an **_ownership mutation gadget_** to change the direct parent of a _mglyph_ tag from a _form_ element in the HTML namespace to a _mtext_ tag in the MathML namespace. This mutation results in the descendants of the _mglyph_ element becoming members of the MathML namespace in the final DOM tree while existing in the HTML namespace in the tree sanitized by DOMPurify.

At the time of this writing, it is still possible to use the _update parser_ feature of the LiveDOM++ tool to reproduce the original bypass using a vulnerable version of DOMPurify (Figure 6: update parser feature, Figure 7: original mXSS bypass).

The details of why and how this works can be referenced in Michal's original article. Understanding the original bypass is important because it provides the foundation for the variation described in this article.

## Building the final payload

Now that I described the building blocks of a potential bypass I will describe the methodology I used to build my bypass. I began by pondering the following:

> Both Michal and Gareth used mutations coming from HTML to MathML and back. What if we throw SVG in the mix?

The picture below illustrates the basic structure of the bypass. Using a nested form **_ownership mutation gadget_** it is possible to change the _mglyph_ tag parent to _mtext;_ that places the descendants of the mglyph tag to be in the MathML namespace in the final DOM tree. Nothing new so far. The difference is that instead of using it to go from HTML to MathML, we are using it to go from SVG to MathML. **_HTML->MathML->HTML->SVG_** to **_HTML->MathML_** to be precise (Figure 8: from SVG to MathML).

At this point, I knew I was on to something but it took me a while to build a working payload. The whole point was figuring out a chain of tags that were safe in the SVG namespace and that contained an XSS payload when transferred to the MathML namespace.

This is what I came up with (Snippet 2: bypass fragment, embedded gist not captured; it is the SVG-side portion of the final payload above: `<mtext><style><path id="</style><imgonerror=alert(1)src>">`).

In the SVG namespace, _mtext_ will be handled as an _unknown_ tag (https://www.w3.org/TR/SVG2/struct.html#UnknownElement). The _style_ tag descendants will be rendered as opposed to its homograph in the HTML namespace, and the _id_ attribute of the SVG _path_ (https://www.w3.org/TR/SVG11/paths.html#PathElement) tag is nothing more than a safe, free-form text identifier.

What happens when this snippet is parsed in the MathML namespace though?

Here is the result (Figure 9: bypass fragment in the MathML namespace).

First of all, _mtext_ is handled as a **MathML text integration point**. This means its descendants will be in the HTML namespace, according to MathML's current specification (https://www.w3.org/TR/MathML3/chapter3.html#presm.mixtextmath).

> ... when MathML is embedded in HTML, or another document markup language, the example is probably best rendered with only the two inequalities represented as MathML at all, **letting the text be part of the surrounding HTML**.

Once in the HTML namespace, _style_ behaves differently and treats everything until its closing tag as raw text. Finally, the malicious _img_ tag is parsed followed by a harmless piece of raw text that reads _">_.

Figure 10: final payload DOM tree before sanitization. Figure 11: final payload DOM tree after sanitization (images).

## Disclosure timeline

I notified cure53 (https://github.com/cure53) on 11/02/2020 around 13:30 (GMT-6). A fix was made available and tested at 13:41 (GMT-6). The fix was officially published in version 2.2.2 (https://github.com/cure53/DOMPurify/releases/tag/2.2.2) sometime later on the very same day.

## Acknowledgments

I want to thank Michal Bentkowski and Gareth Heyes for their incredible work and constant contributions to the security community. Dr.-Ing. Mario Heiderich (@Cure53) for acting so fast and for being a gentleman despite having to stay up late to work on the fix. Lastly, @LiveOverFlow for reminding me in one of his videos (https://www.youtube.com/watch?v=eQFbG6CwwdI) that we need to move away from the basics as soon as we master them and constantly challenge ourselves.
