---
title: "Goodbye innerHTML, Hello setHTML: Stronger XSS Protection in Firefox 148 – Mozilla Hacks - the Web developer blog"
source: "https://hacks.mozilla.org/2026/02/goodbye-innerhtml-hello-sethtml-stronger-xss-protection-in-firefox-148/"
author:
  - "Tom Schuster"
  - "More articles by Tom Schuster…"
  - "More articles by Frederik Braun…"
  - "More articles by Christoph Kerschbaumer…"
published: 2026-02-24
created: 2026-02-28
description: "Cross-site scripting (XSS) remains one of the most prevalent vulnerabilities on the web. The new standardized Sanitizer API provides a straightforward way for web developers to sanitize untrusted HTML before inserting it into the DOM. Firefox 148 is the first browser to ship this standardized security enhancing API, advancing a safer web for everyone. We expect other browsers to follow soon."
tags:
  - "clippings/articles"
  - "_inbox"
---
# Goodbye innerHTML, Hello setHTML: Stronger XSS Protection in Firefox 148 – Mozilla Hacks - the Web developer blog

![](https://hacks.mozilla.org/wp-content/themes/Hax/img/hacks-meta-image.jpg)

> [!summary]+
> > Firefox 148 introduces the new `Sanitizer API` and `setHTML()` method to enhance protection against Cross-site Scripting (XSS) vulnerabilities, which have historically been a top web security concern.
> This API provides a standardized, easy way for developers to sanitize untrusted HTML before inserting it into the DOM, turning malicious HTML into harmless HTML by default.
> Developers can replace error-prone `innerHTML` assignments with `setHTML()` for stronger XSS protections with minimal code changes.
> It offers a default configuration but also allows for custom configurations to define which HTML elements and attributes are allowed or removed.
> The `Sanitizer API` can be combined with `Trusted Types` for even stronger security, centralizing control over HTML parsing and injection.
> This initiative aims to make XSS prevention more accessible to all developers without requiring dedicated security teams or significant architectural changes.

[Cross-Site Scripting (XSS)](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md) remains one of the most prevalent vulnerabilities on the web. The new standardized [Sanitizer API](https://wicg.github.io/sanitizer-api/) provides a straightforward way for web developers to sanitize untrusted HTML before inserting it into the [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model).

The [Sanitizer API](https://wicg.github.io/sanitizer-api/) is designed to help fill that gap by providing a standardized way to turn malicious HTML into harmless HTML — in other words, to sanitize it. The `setHTML( )` method integrates sanitization directly into HTML insertion, providing safety by default. 

Here is an example of sanitizing a simple unsafe HTML:
```html
document.body.setHTML(`<h1>Hello my name is <img src="x" 
onclick="alert('XSS')">`);
```

This sanitization will allow the HTML `<h1>` element while removing the embedded `<img>` element and its `onclick` attribute, thereby eliminating the XSS attack resulting in the following safe HTML:

```html
<h1>Hello my name is</h1>
```

Developers can opt into stronger XSS protections with minimal code changes by replacing error-prone [innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) assignments with [setHTML()](https://developer.mozilla.org/en-US/docs/Web/API/Element/setHTML). 

If the [default configuration](https://wicg.github.io/sanitizer-api/#built-in-safe-default-configuration) of `setHTML( )` is too strict (or not strict enough) for a given use case, developers can provide a [custom configuration](https://developer.mozilla.org/en-US/docs/Web/API/Element/setHTML#options) that defines which HTML elements and attributes should be kept or removed. 

To experiment with the Sanitizer API before introducing it on a web page, we recommend exploring the [Sanitizer API playground](https://sanitizer-api.dev/).