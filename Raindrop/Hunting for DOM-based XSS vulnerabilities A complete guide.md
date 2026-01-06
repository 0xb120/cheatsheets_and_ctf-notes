---
raindrop_id: 1432175860
raindrop_highlights:
  6932b53d2339184fa7c5ea71: da450f21faf1cf9cef583f82f500a78e
  6932b56530d8f2cddafbf154: 4236b0108d4bb55a59202825425d28f8
  6932b5781ff4a506fb137d27: 6644c480ea669a25ed09ee1bb83d18eb
  6932b58356f0e01387ca57e1: 15e1b6cf5cd07635c12f4ad3809242f5
  6932b5b056f0e01387ca675d: 3c0c7dbc92be98b068cfa6e4642d33a7
title: "Hunting for DOM-based XSS vulnerabilities: A complete guide"
description: "null"
source: https://www.intigriti.com/researchers/blog/hacking-tools/exploiting-dom-based-xss-vulnerabilities
created: 1763054371140
type: article
tags:
  - _index
---
# Hunting for DOM-based XSS vulnerabilities: A complete guide

![](https://www.datocms-assets.com/85623/1762158412-intigriti-blog-tools-featured-hacking-for-xxs.png?auto=format&fit=max&w=1200)

> [!summary]
> null





Methodology

Unlike traditional XSS, where you inject your keyword and look for reflection and injection points, DOM-based XSS requires a different approach, whereby you inject your payload into a DOM source and intercept runtime DOM event handlers to find where and if your input is processed. There's also an alternative method for DOM-based XSS vulnerabilities, which involves thoroughly reviewing JavaScript code.
Finding DOM-based XSS via static code analysis

This approach requires a basic foundation of JavaScript. You'll need to manually review JS code, search for DOM sources, and work your way forward until you can find any references to a DOM source.

Additionally, you'll need to review every JavaScript file and code snippet individually, including those that are obfuscated and minified.
Finding DOM-based XSS via DOM runtime interception

This second approach involves actively intercepting DOM events emitted from event handlers in runtime and searching for places where your input may have been processed.
You'll need to make use of your browser's developer console to search for DOM sinks and set up breakpoints where your input may be processed.
Use automated tools!

Both approaches are tedious and can form difficulties when testing at scale. Luckily for us, there are automated tooling available, such as Untrusted Types and DOM Invador that can help you easily spot DOM-based vulnerabilities, including DOM-based cross-site scripting (XSS).