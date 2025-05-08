---
author: "Harel Security Research"
aliases: "ChatGPT Account Takeover - Wildcard Web Cache Deception"
tags: RW_inbox, readwise/articles
url: https://nokline.github.io/bugbounty/2024/02/04/ChatGPT-ATO.html?__readwiseLocation=
date: 2025-04-24
summary: A security researcher discovered a vulnerability in ChatGPT that allowed them to take over user accounts by exploiting a web cache deception issue. By manipulating URLs, the attacker could cache sensitive API endpoints containing user authentication tokens. This flaw was due to inconsistent URL parsing between the CDN and the web server, leading to potential account takeovers.
---
# ChatGPT Account Takeover - Wildcard Web Cache Deception

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

## Highlights


Here’s how I was able to take over your account in ChatGPT. [](https://read.readwise.io/read/01js1gjqvdejfznjc8nzfg49tb)



a web cache deception vulnerability in ChatGPT. The impact of this was critical, as it lead to the leak of user’s auth tokens and subsequently, an account takeover. [](https://read.readwise.io/read/01js1gk5rzyet8pm8m347fs80q)



In this writeup, I will explain how I was able to abuse a path traversal URL parser confusion to achieve what I like to call a “wildcard” cache deception vulnerability [](https://read.readwise.io/read/01js1gkv25kn10kvdhfyxm4qbp)



Additionally, this bug uses a similar concept to the web cache poisoning vulnerability I found in [Glassdoor](https://nokline.github.io/bugbounty/2022/09/02/Glassdoor-Cache-Poisoning.html) last year, which allows us to cache “un-cacheable” files and endpoints. [](https://read.readwise.io/read/01js1gmpc9ee23q64qecgwepwr)



While playing around with ChatGPT’s newly implemented “share” feature [](https://read.readwise.io/read/01js1gnpmncqk4hym0ky56pm3w)



I noticed something weird. None of my shared chat links would update as I continued talking with ChatGPT. [](https://read.readwise.io/read/01js1gnz5y8bjff10ks0myvq9w)



the first thing that came to mind was a caching issue. [](https://read.readwise.io/read/01js1gpcgraw4757fg89n9kkq4)



I checked out the URL, and saw that the path did not have a static extension as expected:
 `https://chat.openai.com/share/CHAT-UUID` [](https://read.readwise.io/read/01js1gq94pk5g9yn1kz5sgy7da)



This meant that there was likely a cache rule that did not rely on the extension of the file, but on its location in the URL’s path. To test this, I checked `https://chat.openai.com/share/random-path-that-does-not-exist` And as expected, it was also cached. [](https://read.readwise.io/read/01js1gr2hgst3yjt5ec2aj1pb3)



cache rule looked something like this: `/share/*` [](https://read.readwise.io/read/01js1grbazdb6f1eef6tdz2ap6)



red flag [](https://read.readwise.io/read/01js1gs5e54vbjzjf6rhsta34c)



relaxed cache rules can be very dangerous, especially with URL parser confusions. [](https://read.readwise.io/read/01js1gs8cz3qeh1bba80dmrqyd)



Path Traversal Confusion
 In a website that uses caching, the request must go through the CDN before it gets to the web server. [](https://read.readwise.io/read/01js1gsh3v6wcx47x8dmfe8yrn)



URL gets parsed twice, which makes it possible for a URL parser confusion. [](https://read.readwise.io/read/01js1gsyr29sd43w0hyzkp4t0p)



Cloudflare’s CDN did NOT decode and did NOT normalize a URL encoded path traversal, but the web server did. [](https://read.readwise.io/read/01js1gtxeg65ac4nppvmsf9sgt)



https://chat.openai.com/share/%2F..%2Fapi/auth/session?cachebuster=123 [](https://read.readwise.io/read/01js1gwc9t195tawb3vjwk2wec)



when the victim goes to `https://chat.openai.com/share/%2F..%2Fapi/auth/session?cachebuster=123`, their auth token will be cached. When the attacker later goes to visit `https://chat.openai.com/share/%2F..%2Fapi/auth/session?cachebuster=123`, they will see the victim’s cached auth token. [](https://read.readwise.io/read/01js1gx0b279vpgq4my0y2ekck)



![](https://nokline.github.io/images/ChatGPT_Attack.svg) [](https://read.readwise.io/read/01js1gxaw3zn68a72gdvdspysc)
