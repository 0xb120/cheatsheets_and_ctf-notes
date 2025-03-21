---
author: "HTTP Toolkit"
aliases: "HTTP/3 Is Everywhere but Nowhere"
tags: RW_inbox, readwise/articles
url: https://httptoolkit.com/blog/http3-quic-open-source-support-nowhere/?__readwiseLocation=
date: 2025-03-21
summary: HTTP/3 has been developed since 2016 and is supported by most browsers and CDNs, but widespread adoption in open-source tools is still lacking. While it offers significant benefits for web traffic, many developers face challenges using it end-to-end in their applications. This gap may create disadvantages for smaller websites compared to larger ones that can take advantage of HTTP/3's improvements.
---
# HTTP/3 Is Everywhere but Nowhere

![rw-book-cover](https://httptoolkit.com/images/header-images/h-sign.jpg)

## Highlights


HTTP/3 has been in development since at least 2016, while QUIC (the protocol beneath it) was first introduced by Google way back in 2013. [](https://read.readwise.io/read/01jpwgfrwyvkassevmgb8pscmg)



Both are now standardized, [**supported in 95% of users' browsers**opens in a new tab](https://caniuse.com/http3), already **used in [32% of HTTP requests to Cloudflareopens in a new tab](https://radar.cloudflare.com/adoption-and-usage)**, and **support is advertised by [35% of websitesopens in a new tab](https://almanac.httparchive.org/en/2024/http#discovering-http3-support)** (through alt-svc or DNS) in the HTTP Archive dataset. [](https://read.readwise.io/read/01jpwgg8r3fdxp3wzkvv1p4ec5)



'm going to assume a basic familiarity with the differences between HTTP/1.1 (et al), HTTP/2 and HTTP/3 here. If you're looking for more context, [http2-explainedopens in a new tab](https://http2-explained.haxx.se/) and [http3-explainedopens in a new tab](https://http3-explained.haxx.se/) from Daniel Stenberg [](https://read.readwise.io/read/01jpwgj9y8pvanmbcfyzntbfbx)

