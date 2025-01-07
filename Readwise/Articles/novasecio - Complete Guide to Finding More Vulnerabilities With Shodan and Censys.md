---
author: novasecio
aliases:
  - Complete Guide to Finding More Vulnerabilities With Shodan and Censys
tags:
  - readwise/articles
url: https://blog.intigriti.com/hacking-tools/complete-guide-to-finding-more-vulnerabilities-with-shodan-and-censys
date: 2024-11-22
---
# Complete Guide to Finding More Vulnerabilities With Shodan and Censys

![rw-book-cover](https://blog.intigriti.com/icon.svg)

## Highlights


**Finding more subdomains using favicons**
 Another way to find subdomains, and other assets that belong to your target is by searching for hosts that have the same favicon hash. Shodan and Censys both provide a native search operator to help you filter the results of hosts that match a specific favicon hash.
 **Shodan:**
` http.favicon.hash:<favicon_hash>`
 **Censys:**
` services.http.response.favicons.hashes:<favicon_hash>`

 **TIP! Want to learn more about how to generate a favicon hash for your target? Check out our small thread on Twitter!**
 ![Tweet](https://pbs.twimg.com/profile_images/1836418823156641792/D22DHqZo.jpg)
 [Intigriti](https://twitter.com/intigriti) [@intigriti](https://twitter.com/intigriti)
 [](https://twitter.com/intigriti/status/1844667511599497453)
 Perform recon with favicon hash to find more targets!
 A small thread! 🧵 👇
 ![](https://pbs.twimg.com/media/GZmTUx5WgAgDOn4.jpg)
 [Posted Oct 11, 2024 at 9:13AM](https://twitter.com/intigriti/status/1844667511599497453)
> [View Highlight](https://read.readwise.io/read/01jda22knkqz1wkqw6x2bwdtp8)

