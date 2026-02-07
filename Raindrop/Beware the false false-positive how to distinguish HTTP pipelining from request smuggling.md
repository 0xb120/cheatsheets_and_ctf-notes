---
raindrop_id: 1321645854
raindrop_highlights:
  69766589a356ca17f3f3b22d: a8ca8ea86c83abac1e98539d569fa448
  697665930a914aeb7ffe2fe4: 5120612c9bf9d812df1234b5e1805ff4
  697665a48aeeeb7cf0a7aba1: 453da16fe41325d61d0bdf979fdaad0a
  697665b0e108164d8b8ff8e3: 0154f3eca5786f51443b3f6e12329eb7
  697665c4e108164d8b90002f: 0db89887e246346b3c6620f2973bab53
  697665d4748fb585ed7f2b3b: e3324f9489615e51182b37e0560f202c
  697665e4b4b57e8e9144264b: 62708651c1fc7d9acafb070ce09074e2
  697665ed6d8be7917df0d3ec: d01bf2b54b3b44401afef4125e7f10c5
  697666278e49ce918d126a03: 2ebfaa4ccea1b2336fde438a7c97b410
  69766629ed10747718ce1ad2: ef5ad5cd5fb597151aba58a034bf6254
  6976663ddac89befc476a718: e7e740712beacb2595ce013c506ab925
  697666e38aeeeb7cf0a8005e: 0934c80bf81f8c51d5a0bd3645128d8b
  697666ea157fccc43ee94cf9: 419ba9a1691d5f6c6a933cdf1b709f1b
  697666feb04c3686ad691a51: a6854818cc3b89f3477342a325fa9533
  6976671ded10747718ce56ce: 69901738c8f938ab68c98f02ebcb6399
  69766726157fccc43ee95c88: 0e302b9c938ca34259e20c834fbe3f06
  6976672fe108164d8b905cc7: 111cff8e5e42ea1186679918c14f3c3f
  697667453f5743acd41737fc: 5fbd433d37bded4399e13c2b0ae478f0
  6976674ddac89befc476ebea: 3393acf5f3a356f807b83a18c463ed2a
  6976675bb04c3686ad6932fb: 4c97d65d2ab33e1eb7a3d623b93e2524
title: "Beware the false false-positive: how to distinguish HTTP pipelining from request smuggling"

description: |-
  Sometimes people think they've found HTTP request smuggling, when they're actually just observing HTTP keep-alive or pipelining. This is usually a false positive, but sometimes there's actually a real

source: https://portswigger.net/research/how-to-distinguish-http-pipelining-from-request-smuggling

created: 2025-08-19
sync-date: 1769635439365
tags:
  - "_index"

 
  - "tech-blog"

---
# Beware the false false-positive: how to distinguish HTTP pipelining from request smuggling

![](https://portswigger.net/cms/images/a5/0d/27b0-twittercard-http_pipelining_twitter.png)

> [!summary]
> Sometimes people think they've found HTTP request smuggling, when they're actually just observing HTTP keep-alive or pipelining. This is usually a false positive, but sometimes there's actually a real





Sometimes people think they've found HTTP request smuggling, when they're actually just observing HTTP keep-alive or pipelining.
This is usually a false positive, but sometimes there's actually a real vulnerability there! In this post I'll explore how to tell the two apart.
Connection reuse false-positives
If you see a request smuggling proof of concept that only works when you reuse connections, it's probably a false positive.
Turbo Intruder PoCs where requestsPerConnection is greater than 1
Burp Intruder when HTTP/1 Connection Reuse is enabled
Burp Repeater when using 'Send group in sequence (single connection)' or 'Enable connection reuse'
Burp Repeater attacks that show two HTTP/1 responses in one
There are three valid closely related vulnerability classes where connection reuse is required:

Connection-locked request smuggling
Connection state attacks (not technically request smuggling)
Client-side desync attacks
So, when creating a request smuggling proof of concept, always disable connection reuse where possible.
To help you distinguish between these two scenarios, I have published a Custom Action called Smuggling or pipelining? - you can install it into Burp Repeater using copy+paste, or import via the Extensibility Helper extension in the BApp store.
HTTP/1 connection reuse
Under the hood, HTTP/1.1 reuses connections by concatenating requests and responses on the underlying TCP/TLS socket. This is known as HTTP connection reuse, pipelining, or keep-alive.
Pipelining is a sub-type of connection reuse where the client sends all their requests in one go and relies on the responses coming back in the correct order.
Connection-locked request smuggling
It would be nice if I could simply say "never reuse connections when testing for request smuggling", but life is never that simple.
Some front-end servers only reuse the upstream connection if the client connection was reused. This means you can end up with request smuggling vulnerabilities that can only be triggered via client-side connection reuse. I call this scenario connection-locked request smuggling.
To confirm this, see if you can send a request over HTTP/2 that triggers a response containing a separate HTTP/1 response nested inside it. This proves it's not a false positive, and means it's worth investing time in trying to build an exploit.
Alternatively, you can often distinguish connection-locked request smuggling using partial requests.
Connection state attacks
When exploring connection-locked request smuggling, you might also uncover connection-state attacks such as first-request routing.

These occur because some servers treat the first request on each connection differently from subsequent requests on the same connection. They are not technically request smuggling vulnerabilities, and can even occur on targets with no front-end server, but ultimately the impact is very similar to connection-locked request smuggling.
Client-side desync attacks
There is one other scenario where connection reuse is exploitable, and that is client-side desync attacks. Note that this comes with a major restriction - the attack request must be something you can get the victims' web browser to send, cross-domain!