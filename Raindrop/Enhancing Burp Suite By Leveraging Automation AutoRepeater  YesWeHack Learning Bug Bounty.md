---
raindrop_id: 1363807548
raindrop_highlights:
  68f68479c712373189485288: aa568e43832c3ceed99f4b8eed90570a
  68f6848e71782e43092511b7: eece03953d0849871a7725b24d8f3c71
  68f68499c712373189485983: 2b0e633767bffe7a6e3072b15122f8d2
  68f684c0b241957228b7d257: f85fedd3da33e34067b4ec5c591ddd0a
  68f684c6413699610a019729: 465853f3a261a520c3c0160bc627f53f
  68f684f047198145301de5ff: 81b18dca277e421b3a278765e42e7db0
  68f684ff5e837d0d98b4337f: a52d2c4f10d709022cf0f16fae6b08a4
  68f6850247198145301de9a3: 79deef51e307d357c148012bd3881c50
  68f68514dd7963dc6b95e190: 151ad2f52bc1be107356a8bffb06f866
title: "Enhancing Burp Suite By Leveraging Automation: AutoRepeater | YesWeHack Learning Bug Bounty"

description: |-
  Discover how AutoRepeater augments your Burp Suite, adding a layer of automation to your bug bounty hunting workflow. Explore the ease of automating repetitive tasks, freeing up your time for more strategic vulnerability hunting.

source: https://www.yeswehack.com/learn-bug-bounty/pimpmyburp-autorepeater-add-automation-burp-suite

created: 1759243613090
type: link
tags:
  - "_index"

 
  - "tech-blog" 
  - "Tools"

---
# Enhancing Burp Suite By Leveraging Automation: AutoRepeater | YesWeHack Learning Bug Bounty

![](https://cdn.sanity.io/images/d51e1jt0/production/8f2a50935461bc0ccb3e708cb5d22c8864f3c263-1940x935.webp)

> [!summary]
> Discover how AutoRepeater augments your Burp Suite, adding a layer of automation to your bug bounty hunting workflow. Explore the ease of automating repetitive tasks, freeing up your time for more strategic vulnerability hunting.





How to use AutoRepeater?

The interface is pretty clear
Replacements Tab:
Global Replacement: request header, body, param name, but also, cookie name, cookie value, string, first line, remove parameter by name, by value, matching param name and replace value…
Replacements: same options as global replacement
Conditional replacements: boolean operators, multiple match type (domain name, HTTP method, protocol, file extension, URL, sent from tool… and many more)
Logs Tab:
Log Filter: add a filter to show only interesting requests with a specific behavior in your testing process (filter can be applied on URL, request header, request body, file extension…)
Log Highlighter: adding colors on specific requests which match with a string, request, param, extension, HTTP method…
Left Side:
Your requests
Replayed requests
Bottom:
Request & Response viewer with original and modified request
Diff between (for requests and responses)
Be as creative as possible, here are examples of which rules can be created:

Add new headers (x-forwarded-for, x-forwarded-host, origin…),
Remove authentication header or cookie (JWT for example),
Replace value like user to admin, false to true,
Add JSON parameters,
Change HTTP Method (PUT to POST or vice versa),
Add GET or POST parameters,
Add .old or .back to files extension (.php.old, .jsp.old…),
Remove CSRF token,
Math parameters for Open Redirect, reflected XSS or SSRF
Try to bypass “403 Forbidden” by adding specific headers or things like “.;/” in URL
Chain AutoRepeater with Hackvertor extension
And many more…
To keep in mind: for each replacement configured in AutoRepeater, a new request will be generated, this can therefore quickly become unreadable. Luckily, there is a feature to highlight requests, and this is what we will see right now.
Log Highlighter & Log Filter
AutoRepeater has a “Log Highlighter” feature which can be used with conditional match, to highlight only what you exactly want. For example, only match “200 OK” status or if HTTP method is “PUT” or “POST”.