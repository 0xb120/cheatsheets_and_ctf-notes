---
author: blackbird-eu
aliases:
  - "Discovering Hidden Parameters: An Advanced Guide"
tags:
  - RW_inbox
  - readwise/articles
url: https://www.intigriti.com/researchers/blog/hacking-tools/finding-hidden-input-parameters?__readwiseLocation=
date: 2025-07-07
---
# Discovering Hidden Parameters: An Advanced Guide

![rw-book-cover](https://blog.intigriti.com/icon.svg)

## Highlights


1. HTML input fields
 The most straightforward way to discover more parameters is by scraping the **'id'** and the **'name'** attributes of every HTML element, including non-form input fields. This is often done by crawling the target and parsing the attribute values. Automated tools like GoSpider and GetAllParams (GAP) [](https://read.readwise.io/read/01jwxc8z0z0btakbm7mm7drftm)



2. JavaScript file enumeration
 We all know that [JavaScript files](https://www.intigriti.com/researchers/blog/hacking-tools/testing-javascript-files-for-bug-bounty-hunters) are a goldmine for bug bounty hunters as they contain several references to API endpoints, app routes and of course input parameters. Examining JavaScript files and looking for specific function calls that parse and retrieve values of parameters is key here. [](https://read.readwise.io/read/01jwxca8m66tcf76basr9aacw9)



Variable names
 Besides only looking at potential parameters in pre-defined HTTP requests or parsing functions, you can also try and collect a list of variable names and request them as body or query parameters! [](https://read.readwise.io/read/01jwxcbgw2whsdp7g56a5c703s)



Intercepting client-side parameters
 Another way to detect new parameters is to intercept them on the client-side. Tools like Eval Villain allow you to intercept and get notified if any of the supplied input parameters are processed by the DOM. [](https://read.readwise.io/read/01jwxcc1d2ndxp707g24snnz9s)



3. Google/GitHub/Wayback Machine enumeration
 [Google dorking](https://www.intigriti.com/researchers/blog/hacking-tools/google-dorking-for-beginners-how-to-find-more-vulnerabilities-using-google-search) is another method to perform reconnaissance and find more interesting URLs, including parameters linked to your target. [](https://read.readwise.io/read/01jwxccg6x4q36zsf86nxtfbf3)



A simple search could yield you 10s of new potential parameters that you can try out when testing:
 site:.example.com inurl:? || inurl:& [](https://read.readwise.io/read/01jwxccx6rbt4ge6m4gw5xjaga)



Getting more advanced using the Internet Archive
 Internet Archive (also referred to as Wayback Machine) can help you discover even more parameters as it indexes multiple versions of almost all web pages on the internet. [](https://read.readwise.io/read/01jwxcd9fvht07t4geq6fj1bwp)



4. Parameter fuzzing
 Parameter fuzzing is the most accurate and scalable way to discover hidden and unreferenced parameters. By observing response changes possibly induced by a parameter, you can accurately get an overview if a certain parameter is processed by the backend. [](https://read.readwise.io/read/01jwxcdn95fpj8wnhrma9bf90c)



multiple tools available to automate this task, such as Ffuf, Arjun, x8, ParamMiner, etc. [](https://read.readwise.io/read/01jwxcdyegtry3ngd4fd6z6xpn)



5. Re-use of parameters
 In some instances, query and body parameters are re-used and accepted throughout multiple application routes and API endpoints. Both ZAProxy and Burp Suite provide you with a crawl history that you can easily export and parse. [](https://read.readwise.io/read/01jwxcenx87symbhvmb3rc45t5)

