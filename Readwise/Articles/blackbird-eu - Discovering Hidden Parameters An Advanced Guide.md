---
author: blackbird-eu
aliases:
  - "Discovering Hidden Parameters: An Advanced Guide"
tags:
  - readwise/articles
url: https://www.intigriti.com/researchers/blog/hacking-tools/finding-hidden-input-parameters?__readwiseLocation=
created: 2025-07-07
---
# Discovering Hidden Parameters: An Advanced Guide

![rw-book-cover](https://blog.intigriti.com/icon.svg)

## Highlights


### HTML input fields

The most straightforward way to discover more parameters is by scraping the `id` and the `name` attributes of every HTML element, including non-form input fields. This is often done by crawling the target and parsing the attribute values. Automated tools like `gospider`, [fallparams](../../Raindrop/GitHub%20-%20ImAyrixfallparams%20Find%20All%20Parameters%20-%20Tool%20to%20crawl%20pages,%20find%20potential%20parameters%20and%20generate%20a%20custom%20target%20parameter%20wordlist.md), [GAP](../../Dev,%20ICT%20&%20Cybersec/Tools/Burpsuite.md#GAP%20✅), and every other crawler/spider in [Endpoint Crawling](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/HTTP%20Recon%20and%20Enumeration.md#Endpoint%20Crawling) can help.[](https://read.readwise.io/read/01jwxc8z0z0btakbm7mm7drftm)
### JavaScript file enumeration

We all know that [JavaScript files](https://www.intigriti.com/researchers/blog/hacking-tools/testing-javascript-files-for-bug-bounty-hunters) are a goldmine for bug bounty hunters as they contain several references to API endpoints, app routes and of course input parameters. 

Examining JavaScript files and looking for specific function calls that parse and retrieve values of parameters is key here. [](https://read.readwise.io/read/01jwxca8m66tcf76basr9aacw9)

### Variable names
Besides only looking at potential parameters in pre-defined HTTP requests or parsing functions, you can also try and **collect a list of variable names and request them as body or query parameters**! [](https://read.readwise.io/read/01jwxcbgw2whsdp7g56a5c703s)

If you see:
```js
var siteKey = "asUgXlsLs"
```

Try: `https://example.com?siteKey=":alert(0)//`

### Intercepting client-side parameters

Another way to detect new parameters is to **intercept them on the client-side**. Tools like Eval Villain allow you to intercept and get notified if any of the supplied input parameters are processed by the DOM. [](https://read.readwise.io/read/01jwxcc1d2ndxp707g24snnz9s)

### Google/GitHub/Wayback Machine enumeration

[Google dorking](https://www.intigriti.com/researchers/blog/hacking-tools/google-dorking-for-beginners-how-to-find-more-vulnerabilities-using-google-search) is another method to perform reconnaissance and find more interesting URLs, including parameters linked to your target. [](https://read.readwise.io/read/01jwxccg6x4q36zsf86nxtfbf3)

A simple search could yield you 10s of new potential parameters that you can try out when testing:
```txt
site:.example.com inurl:? || inurl:& 
```
[](https://read.readwise.io/read/01jwxccx6rbt4ge6m4gw5xjaga)

### Getting more advanced using the Internet Archive

Internet Archive (also referred to as Wayback Machine) can help you discover even more parameters as it indexes multiple versions of almost all web pages on the internet. [](https://read.readwise.io/read/01jwxcd9fvht07t4geq6fj1bwp)

### Parameter fuzzing

Parameter fuzzing is the most accurate and scalable way to discover hidden and unreferenced parameters. By observing response changes possibly induced by a parameter, you can accurately get an overview if a certain parameter is processed by the backend. [](https://read.readwise.io/read/01jwxcdn95fpj8wnhrma9bf90c)

Multiple tools available to automate this task, such as [ffuf](../../Dev,%20ICT%20&%20Cybersec/Tools/ffuf.md), Arjun, x8, [Param Miner ✅](../../Dev,%20ICT%20&%20Cybersec/Tools/Burpsuite.md#Param%20Miner%20✅), etc. [](https://read.readwise.io/read/01jwxcdyegtry3ngd4fd6z6xpn)

### Re-use of parameters

In some instances, query and body parameters are re-used and accepted throughout multiple application routes and API endpoints. Both ZAProxy and Burp Suite provide you with a crawl history that you can easily export and parse. [](https://read.readwise.io/read/01jwxcenx87symbhvmb3rc45t5)

