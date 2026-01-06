---
author: jakearchibald.com
aliases:
  - How to Win at CORS
tags:
  - readwise/articles
url: https://jakearchibald.com/2021/cors/?__readwiseLocation=
created: 2025-03-07
---
# How to Win at CORS

[Cross-origin resource sharing (CORS)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-origin%20resource%20sharing%20(CORS).md) is hard. It's hard because it's part of how browsers fetch stuff, and that's a set of behaviours that started with the very first web browser [](https://read.readwise.io/read/01jnre9aqw1vvhhp15pwe1j36g).

To make things *interactive*, I built an exciting new app: you can dive right into [the playground](https://jakearchibald.com/2021/cors/playground/) now [](https://read.readwise.io/read/01jnrearfx5zknx5sfh5av1znb). ^3625b2

## CORS without CORS

Browsers have been able to include images from other sites for almost 30 years. You don't need the other site's permission to do this, you can just do it. And it didn't stop with images:
```html
 <script src="…"></script>
 <link rel="stylesheet" href="…" />
 <iframe src="…"></iframe>
 <video src="…"></video>
 <audio src="…"></audio> [](https://read.readwise.io/read/01jnrmhb84ptpxckra0nkxn1nh)
```

## Origins vs Sites

Checker at https://jakearchibald.com/2021/cors/?__readwiseLocation=#origins-vs-sites
#tools 

But how does the browser know that `https://help.yourbank.com` and `https://profile.yourbank.com` are part of the same site, but `https://yourbank.co.uk` and `https://jakearchibald.co.uk` are different sites? I mean… they all have three parts separated by dots.

 Well, the answer was a bunch of heuristics in each browser, but in 2007 Mozilla swapped their heuristics for a list. That list is now maintained as a separate community project known as the [public suffix list](https://publicsuffix.org/), and it's used by all browsers and many other projects. [](https://read.readwise.io/read/01jnrmznp4jks61meqvxyspaj5)


So `https://app.jakearchibald.com` and `https://other-app.jakearchibald.com` are part of the same site, but `https://app.glitch.me` and `https://other-app.glitch.me` are different sites. [](https://read.readwise.io/read/01jnrn231mzgdncnbpeh6zws3d)

These cases are different because `glitch.me` is on the public suffix list whereas `jakearchibald.com` is not. This is 'correct', because different people 'own' the subdomains of `glitch.me`, whereas I own all the subdomains of `jakearchibald.com`. [](https://read.readwise.io/read/01jnrn2pnh6zwqz58ce9x5zy13)

## Making a CORS request

There's no easy rule for what does and doesn't require CORS. For example:
```html
 <!-- Not a CORS request -->
 <script src="https://example.com/script.js"></script>
 <!-- CORS request -->
 <script type="module" src="https://example.com/script.js"></script>
```
 The best way to figure it out is to try it and look at network DevTools. In Chrome and Firefox, cross-origin requests are sent with a [`Sec-Fetch-Mode` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Mode) which will tell you if it's a CORS request or not. [](https://read.readwise.io/read/01jnrn93rcpz8rar65x7syz308)

## CORS responses

A valid value gives the other origin access to the response body, and also a subset of the headers:
 - `Cache-Control`
 - `Content-Language`
 - `Content-Length`
 - `Content-Type`
 - `Expires`
 - `Last-Modified`
 - `Pragma`
 The response can include another header, `Access-Control-Expose-Headers`, to reveal additional headers:
 
 `Access-Control-Expose-Headers: Custom-Header-1, Custom-Header-2`
 
 The matching is case-insensitive since header names are case-insensitive. You can also use:
 
 `Access-Control-Expose-Headers: *`
 
 …to expose (almost) all the headers [](https://read.readwise.io/read/01jnrndzatdftbnnd8qf6n2z66)

The `Set-Cookie` and `Set-Cookie2` (a deprecated failed 'sequel' to `Set-Cookie`) headers are never exposed, to avoid leaking cookies across sites. [](https://read.readwise.io/read/01jnrne7bxn7gashfna5h8tgze)

## Unusual requests and pre-flight

If the request method isn't `GET`, `HEAD`, or `POST`, or it includes headers or header values that aren't part of the [safelist](https://fetch.spec.whatwg.org/#cors-safelisted-request-header), then it counts as unusual. [](https://read.readwise.io/read/01jnrnk9d7a9dacvrzqj8xm4rw)

> Note: pre-flight check

- `Access-Control-Max-Age` - The number of seconds to cache this preflight response, to avoid the need for further preflights to this URL. The default is 5 seconds. Some browsers have an upper-limit on this. In Chrome it's 600 (10 minutes), and in Firefox it's 86400 (24 hours).
 - `Access-Control-Allow-Methods` - The *unusual* methods to allow. This can be a comma-separated list, and values are case-sensitive. If the main request is to be sent without credentials, this can be `*` to allow (almost) any method. You can't allow `CONNECT`, `TRACE`, or `TRACK` as these are on a [🔥💀 FORBIDDEN LIST 💀🔥](https://fetch.spec.whatwg.org/#forbidden-method) for security reasons.
 - `Access-Control-Allow-Headers` - The *unusual* headers to allow. This can be a comma-separated list, and values are case-insensitive since header names are case-insensitive. If the main request is to be sent without credentials, this can be `*` to allow any header that isn't on a [🔥💀 DIFFERENT FORBIDDEN LIST 💀🔥](https://fetch.spec.whatwg.org/#forbidden-header-name).
 
 Headers in the [🔥💀 FORBIDDEN LIST 💀🔥](https://fetch.spec.whatwg.org/#forbidden-header-name) are headers that must remain in the browser's control for security reasons. They're automatically (and silently) stripped from CORS requests and `Access-Control-Allow-Headers`. [](https://read.readwise.io/read/01jnrnpfrvd5mgbhz508qvkmbr)

Let's put all of that together, for one last time, in the CORS playground:
 - [A simple request](https://jakearchibald.com/2021/cors/playground/?prefillForm=1&requestMethod=GET&requestUseCORS=1&requestSendCredentials=&preflightStatus=206&preflightAllowOrigin=&preflightAllowCredentials=&preflightAllowMethods=&preflightAllowHeaders=&responseAllowOrigin=*&responseAllowCredentials=&responseExposeHeaders=). This doesn't require a preflight.
 - [An unusual header](https://jakearchibald.com/2021/cors/playground/?prefillForm=1&requestMethod=GET&requestUseCORS=1&requestSendCredentials=&preflightStatus=405&preflightAllowOrigin=&preflightAllowCredentials=&preflightAllowMethods=&preflightAllowHeaders=&responseAllowOrigin=*&responseAllowCredentials=&responseExposeHeaders=&requestHeaderName=hello&requestHeaderValue=world). This triggers a preflight, and the server doesn't allow the request.
 - [An unusual header, again](https://jakearchibald.com/2021/cors/playground/?prefillForm=1&requestMethod=GET&requestUseCORS=1&requestSendCredentials=&preflightStatus=206&preflightAllowOrigin=*&preflightAllowCredentials=&preflightAllowMethods=&preflightAllowHeaders=*&responseAllowOrigin=*&responseAllowCredentials=&responseExposeHeaders=&requestHeaderName=hello&requestHeaderValue=world), but this time the preflight is correctly configured, so the request goes through.
 - [A normal `Range` header](https://jakearchibald.com/2021/cors/playground/?prefillForm=1&requestMethod=GET&requestUseCORS=1&requestSendCredentials=&preflightStatus=206&preflightAllowOrigin=*&preflightAllowCredentials=&preflightAllowMethods=&preflightAllowHeaders=*&responseAllowOrigin=*&responseAllowCredentials=&responseExposeHeaders=&requestHeaderName=range&requestHeaderValue=bytes%3D0-). This relates to the [spec change I made](https://github.com/whatwg/fetch/pull/1312). When browsers implement the change, this request won't need a preflight. It's currently implemented in Chrome Canary.
 - [An unusual method](https://jakearchibald.com/2021/cors/playground/?prefillForm=1&requestMethod=Wibbley-Wobbley&requestUseCORS=1&requestSendCredentials=&preflightStatus=206&preflightAllowOrigin=*&preflightAllowCredentials=&preflightAllowMethods=Wibbley-Wobbley&preflightAllowHeaders=&responseAllowOrigin=*&responseAllowCredentials=&responseExposeHeaders=). This highlights the Chrome bug documented above. The request won't go through in Chrome, but it'll work in other browsers.
 - [An unusual method, again](https://jakearchibald.com/2021/cors/playground/?prefillForm=1&requestMethod=Wibbley-Wobbley&requestUseCORS=1&requestSendCredentials=&preflightStatus=206&preflightAllowOrigin=*&preflightAllowCredentials=&preflightAllowMethods=Wibbley-Wobbley%2C+WIBBLEY-WOBBLEY&preflightAllowHeaders=&responseAllowOrigin=*&responseAllowCredentials=&responseExposeHeaders=). This works around the Chrome bug. [](https://read.readwise.io/read/01jnrnrwfzcy9ecgv3jjenyjvc)

