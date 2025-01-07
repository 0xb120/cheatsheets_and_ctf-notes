---
author: "Paul Gerste"
aliases: "Never Underestimate CSRF: Why Origin Reflection Is a Bad Idea"
tags: RW_inbox, readwise/articles
url: https://www.sonarsource.com/blog/never-underestimate-csrf-why-origin-reflection-is-a-bad-idea/
date: 2024-12-13
---
# Never Underestimate CSRF: Why Origin Reflection Is a Bad Idea

![rw-book-cover](https://assets-eu-01.kc-usercontent.com:443/7630306f-9a2f-018d-2726-3ef76ef712f4/c9a1ac0a-f9cc-4bf0-9154-3d6de380a8d2/Never%20Underestimate%20CSRF_Blog-landscape.png)

## Highlights


The vulnerability ([CVE-2024-55500](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-55500)) is a Cross-Site Request Forgery (CSRF) issue caused by a CORS misconfiguration.
> [View Highlight](https://read.readwise.io/read/01jexkd8s0r88t245h3ms7m7qd)



![](https://assets-eu-01.kc-usercontent.com:443/f42196a6-70a1-01d0-99f1-43134f12a58b/5b794a56-47bf-4e45-a0d9-2301a7c49b9b/whistle-issue.png)
> [View Highlight](https://read.readwise.io/read/01jexkev6xdjy07vmzrk13drz1)



What is highlighted here is a potential Cross-Origin Resource Sharing (CORS) configuration issue. CORS can be used by a webserver to allow other websites to interact with it. This is done by setting special `Access-Control-*` HTTP response headers that the browser will abide by.
> [View Highlight](https://read.readwise.io/read/01jexkf12vt9qeetcg47t9z1qk)



In the highlighted code snippet, we can see that the request's `Origin` header is reflected in the response's `Access-Control-Allow-Origin` header. This is unsafe because it essentially enables CORS for all origins, allowing any website to send requests to the server and read the response!
> [View Highlight](https://read.readwise.io/read/01jexkfbe8vyyamwsemmwc5bw8)



If sensitive information is contained in any response, malicious websites can read it when the user visits them. Similarly, the malicious website can also control what data is included in the request, resulting in a Cross-Site Request Forgery (CSRF) vulnerability.
> [View Highlight](https://read.readwise.io/read/01jexkgdfvfds4a55dggwcw98t)



CORS controls not only the origins but also the data that can be contained in requests. More specifically, the `Access-Control-Allow-Headers` response header tells the browser which headers can be sent in the cross-origin request.
> [View Highlight](https://read.readwise.io/read/01jexkhy6rww4a0djcrramrc7h)



To send a form-encoded body, the attacker would need to set a `Content-Type` header, this time with the value of `application/x-www-form-urlencoded`. But shouldn't this also fail since `Content-Type` is not in the server's `Access-Control-Allow-Headers`?
 In this case, the attacker is lucky because of the [Simple Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests) concept. Simple Requests can be sent cross-origin without the need for CORS. Similarly, parts of Simple Requests, such as [safelisted request headers](https://fetch.spec.whatwg.org/#simple-header), can be included in cross-origin requests, even when CORS did not specifically negotiate them. This includes the `Content-Type` header when its value is either `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`.
> [View Highlight](https://read.readwise.io/read/01jexkjzzxwttayvag4bxbj999)



This means the attacker can send a CORS request with `Content-Type: application/x-www-form-urlencoded` even though the server does not explicitly allow the `Content-Type` header!
> [View Highlight](https://read.readwise.io/read/01jexkkpck36k0ws1xzt028nrq)



We already talked about the `Content-Type` header in Simple Requests, but what else does a request need to be considered simple? There are several minor requirements, but the important one for our case is that a Simple Request can only use the `GET`, `POST`, or `HEAD` methods
> [View Highlight](https://read.readwise.io/read/01jexkn75et7x1f9ph4ncddnvr)



![](https://assets-eu-01.kc-usercontent.com:443/f42196a6-70a1-01d0-99f1-43134f12a58b/669482f5-37bf-4a6c-b053-8a86016b5d6a/Whistle%20Attack%20Flow.png)
> [View Highlight](https://read.readwise.io/read/01jexknqafvdqww7w5rk3wm8kx)



as we learned earlier, Simple Requests can still be used to communicate with Whistle's API because they don't require CORS. To fix this issue, we suggested checking the `Sec-Fetch-Site` header to identify cross-origin requests and block them:
 if (req.headers['sec-fetch-site'] !== 'same-origin') {
 return res.status(403).end('Forbidden');
 }
 This header is a so-called [forbidden header,](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name) which is set by the browser and cannot be changed by the page. It will only have the value `same-origin` when the requesting page is of the same origin as the destination URL, making it a good solution for this problem.
> [View Highlight](https://read.readwise.io/read/01jexkpd08yp6zp1ppwxznmsbe)



