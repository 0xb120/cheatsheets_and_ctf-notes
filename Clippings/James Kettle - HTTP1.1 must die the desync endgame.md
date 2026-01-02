---
title: "HTTP/1.1 must die: the desync endgame"
source: https://portswigger.net/research/http1-must-die
author:
  - James Kettle
published: 2025-08-06
created: 2025-08-11
description: Abstract Upstream HTTP/1.1 is inherently insecure and regularly exposes millions of websites to hostile takeover. Six years of attempted mitigations have hidden the issue, but failed to fix it. This p
tags: [clippings/articles]
aliases: [0.CL desync attacks, expect-based desync attacks]
---
# HTTP/1.1 must die: the desync endgame

> [!summary]
> This paper argues that HTTP/1.1 is fundamentally insecure due to weak request boundaries and multiple ways to specify message length, leading to widespread HTTP desync attacks and hostile takeovers. Despite six years of attempted mitigations, these issues persist, often hidden or bypassed by attackers.
> The author introduces novel HTTP desync attack classes, including 0.CL desyncs (overcoming connection deadlocks with 'early-response gadgets' like the `/con` quirk on IIS) and Expect-based desyncs. These techniques were demonstrated through critical case studies, compromising core infrastructure within Cloudflare (affecting 24 million websites), Akamai (affecting LastPass and potentially example.com), Netlify CDN, T-Mobile, and GitLab, yielding over $350,000 in bug bounties.
> The research highlights that current mitigations (e.g., WAFs, regexes) are insufficient and often make systems more complex and vulnerable. The author provides the open-source Burp Suite extension \\"HTTP Request Smuggler v3.0\\" for systematic detection of parser discrepancies.
> The ultimate solution, the paper concludes, is the widespread adoption of upstream HTTP/2+, a binary protocol that eliminates the root cause of these desync vulnerabilities. Until then, organizations using HTTP/1.1 are advised to enable normalization, validation, avoid niche web servers, and disable upstream connection reuse.

PDF version: [US-25-Kettle-HTTP1-Must-Die-The-Desync-Endgame-wp](attachments/US-25-Kettle-HTTP1-Must-Die-The-Desync-Endgame-wp.pdf)

## The desync endgame (TL;DR)

Upstream HTTP/1.1 is inherently insecure and regularly exposes millions of websites to hostile takeover.

This paper introduces several novel classes of HTTP desync attack capable of mass compromise of user credentials.

Ultimately, I argue that [HTTP request smuggling](https://portswigger.net/web-security/request-smuggling) must be recognized as a fundamental protocol flaw. The past six years have demonstrated that addressing individual implementation issues will never eliminate this threat. Although my findings have been reported and patched, websites remain silently vulnerable to inevitable future variants.

### The fatal flaw in HTTP/1.1

HTTP/1.1 has a fatal, highly-exploitable flaw - the boundaries between individual HTTP requests are very weak. Requests are simply concatenated on the underlying TCP/TLS socket with no delimiters, and there are multiple ways to specify their length. This means attackers can create extreme ambiguity about where one request ends and the next request starts. Major websites often use reverse proxies, which funnel requests from different users down a shared connection pool to the back-end server. This means that an attacker who finds the tiniest parser discrepancy in the server chain can cause a desync, apply a malicious prefix to other users' requests, and usually achieve complete site takeover:

In this paper, we'll use the following acronyms for the four major length interpretations:

```
CL (Content-Length) 
TE (Transfer-Encoding) 
0 (Implicit-zero) 
H2 (HTTP/2's built-in length)
```

HTTP/1.1 may look secure at first glance because if you apply the original request smuggling methodology and toolkit, you'll have a hard time causing a desync. But why is that?

These days, the probe will probably fail even if your target is actually vulnerable, for one of three reasons:

- WAFs now use regexes to detect and block requests with an obfuscated Transfer-Encoding header, or potential HTTP requests in the body.
- The /robots.txt detection gadget doesn't work on your particular target.
- There's a server-side race condition which makes this technique highly unreliable on certain targets.

The alternative, timeout-based detection strategy discussed in my previous research is also heavily fingerprinted and blocked by WAFs.

### "HTTP/1 is simple" and other lies

There's a widespread, dangerous misconception that HTTP/1.1 is a robust foundation suitable for any system you might build.

The moment you attempt to proxy HTTP/1.1, it becomes a lot less simple.

Here are five lies that will be critical to a real-world exploit discussed later in this paper
- Lie 1: An HTTP/1.1 request can't directly target an intermediary
- Lie 2: An HTTP/1.1 desync can only be caused by a parser discrepancy
- Lie 3: An HTTP/1.1 response contains everything a proxy needs to parse it
- Lie 4: An HTTP/1.1 response can only contain one header block
- Lie 5: A complete HTTP/1.1 response requires a complete request

More desync attacks are always coming.

## A strategy to win the desync endgame

### Detecting parser discrepancies

we need a detection strategy that reliably identifies the underlying flaws that make desync attacks possible, rather than attempting brittle attacks with many moving parts.

Burp Suite extension [HTTP Request Smuggler ✅](../Dev,%20ICT%20&%20Cybersec/Tools/Burpsuite.md#HTTP%20Request%20Smuggler%20✅). Here's a high-level overview of the three key elements used for analysis, and the possible outcomes:

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-1.png)
### Understanding V-H and H-V discrepancies

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-2.png)

Here, HTTP Request Smuggler has detected that sending a request with a partially-hidden Host header causes a unique response that can't be triggered by sending a normal Host header, or by omitting the header entirely, or by sending an arbitrary masked header. 

This is strong evidence that there's a parser discrepancy in the server chain used by the target. If we assume there's a front-end and a back-end, there's two key possibilities:
- **Visible-Hidden (V-H)**: The masked Host header is visible to the front-end, but hidden from the back-end
	- [TE.CL smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/TE.CL%20smuggling%20vulnerabilities.md)
	- [CL.0 smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md)
- **Hidden-Visible (H-V)**: The masked Host header is hidden from the front-end, but visible to the back-end
	- [CL.TE smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.TE%20smuggling%20vulnerabilities.md)
	- [H2.TE smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/H2.TE%20smuggling%20vulnerabilities.md)
	- [0.CL desync attacks](James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame.md#0.CL%20desync%20attacks)

You can often distinguish between V-H and H-V discrepancies by paying close attention to the responses, and guessing whether they originated from a front-end or back-end.

>[!note]
>Note that the specific status codes are not relevant, and can sometimes be confusing. All that matters is that they're different.

### Turning a V-H discrepancy into a CL.0 desync

Given a V-H discrepancy, you could attempt a [TE.CL smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/TE.CL%20smuggling%20vulnerabilities.md) by hiding the Transfer-Encoding header from the back-end, or try a [CL.0 smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md) exploit by hiding the Content-Length header.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-3.png)

On a different target, the above exploit failed because the front-end server was rejecting GET requests that contained a body. I was able to work around this simply by switching the method to OPTIONS.

### Detection strategies

By combining different headers, permutations, and strategies, the tool achieves superior coverage.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-4.png)

This target was once again straightforward to exploit using a [CL.0 smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md).

### Detecting high-risk parsing

Discrepancy-detection approach can also identify servers that deviate from accepted parsing conventions and are, therefore, likely to be vulnerable if placed behind a reverse proxy.

Request:
```http
POST / HTTP/1.1\r\n
Content-Length: 22\r\n
A: B\r\n
\nExpect: 100-continue\r\n
```

Response:
```http
HTTP/1.1 100 Continue 

HTTP/1.1 302 Found 
Server: <redacted>
```

### Exploiting H-V on IIS behind ALB

Large number of vulnerable systems using Microsoft IIS behind AWS Application Load Balancer (ALB).

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-5.png)

As you can infer from the server banners, this is a **H-V discrepancy**: when the malformed Host header is obfuscated, ALB doesn't see it and passes the request through to the back-end server.

The classic way to exploit a H-V discrepancy is with a [CL.TE smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.TE%20smuggling%20vulnerabilities.md) desync, as the Transfer-Encoding header usually takes precedence over the Content-Length

Thomas Stacey [independently discovered it](https://assured.se/posts/the-single-packet-shovel-desync-powered-request-tunnelling), and bypassed Desync Guardian using an [H2.TE smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/H2.TE%20smuggling%20vulnerabilities.md) desync.

### Exploiting H-V without Transfer-Encoding

I discovered a H-V discrepancy on a certain website which blocks all requests containing Transfer-Encoding, making CL.TE attacks impossible. There was only one way forward with this: a [0.CL desync attacks](James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame.md#0.CL%20desync%20attacks).

## 0.CL desync attacks

### The 0.CL deadlock

0.CL desync attacks are widely regarded as unexploitable.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-6.png)

The front-end doesn't see the Content-Length header, so it will regard the orange payload as the start of a second request. This means it buffers the orange payload, and only forwards the header-block to the back-end:

The back end does see the Content-Length header, so it will wait for the body to arrive. Meanwhile, the front-end will wait for the back-end to reply. Eventually, one of the servers will time out and reset the connection

In essence, 0.CL desync attacks usually result in an upstream connection deadlock.

### Breaking the 0.CL deadlock

Whenever I tried to use the [single-packet attack](https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local) on a static file on a target running nginx, nginx would respond to the request before it was complete. This hinted at a way to make 0.CL exploitable.

The key to escaping the 0.CL deadlock is to **find an early-response gadget**: a way to make the back-end server respond to a request without waiting for the body to arrive. 

This is straightforward on nginx, but my target was running IIS, and the static file trick didn't work there. So, how can we persuade IIS to respond to a request without waiting for the body to arrive? 

> Do not use the following reserved names for the name of a file:
> CON, PRN, AUX, NUL, COM1, COM2, COM3, COM4, COM5, COM6, COM7...

If you try to access a file or folder using a reserved name, the operating system will throw an exception for amusing legacy reasons. We can make a server hit this quirk simply by requesting 'con' inside any folder that's mapped to the filesystem.

if I hit `/con` on the target website, IIS would respond without waiting for the body to arrive, and helpfully leave the connection open

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-7.png)

On other servers, I found server-level redirects operated as early-response gadgets. However, I never found a viable gadget for Apache; they're too studious about closing the connection when they hit an error condition.

### Moving beyond 400 Bad Request

To prove you've found a 0.CL desync, the next step is to **trigger a controllable response**. After the attack request, send a 'victim' request containing a second path nested inside the header block:

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-8.png)

If you set the Content-Length of the first request correctly, **it will slice the initial bytes off the victim request**, and you'll see a response indicating that the hidden request line got processed.

This is sufficient to prove there's a 0.CL desync, but it's obviously not a realistic attack

We need a way to add our payload to the victim's request. **We need to convert our 0.CL into a [CL.0 smuggling vulnerabilities](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md).**

### Converting 0.CL into CL.0 with a double-desync

To convert 0.CL into [CL.0 desync](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md), we need a **double-desync**! This is a multi-stage attack where the attacker uses a sequence of two requests to set the trap for the victim:

1. The first request poisons the connection with a 0.CL desync
2. The poisoned connection weaponises the second request into a CL.0 desync, which then repoisons the connection with a malicious prefix
3. The malicious prefix then poisons the victim's request, causing a harmful response

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-9.png)

The cleanest way to achieve this would be **to have the 0.CL cut the entire header block off the first request**:

```http
POST /nul HTTP/1.1
Content-length:
 163
```

```http
POST / HTTP/1.1
Content-Length: 111

GET / HTTP/1.1
Host: <redacted>

GET /wrtz HTTP/1.1
Foo: bar
```

This is not as easy as it looks. You need to know the exact size of the second request header block, and virtually all front-end servers append extra headers. 

Most servers **insert headers at the end of the header block**, not at the start. So, if our smuggled request starts before that, the attack will work reliably! Here's an example that uses an input reflection to reveal the inserted header:

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-10.png)

From this point, we can use traditional CL.0 exploit techniques. On this target, I used the HEAD technique to serve malicious JavaScript to random users:

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-11.png)

You can experiment with this technique yourself for free using our new Web Security Academy lab [0.CL Request Smuggling](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-0cl-request-smuggling).

## Expect-based desync attacks

### The Expect complexity bomb

The Expect header is an ancient optimisation that splits sending a single HTTP request into a two-part process. The client sends the header block containing `Expect: 100-continue`, and the server evaluates whether the request would be accepted. If the server responds with HTTP/1.1 100 Continue, the client is then permitted to send the request body.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-12.png)

Expect breaks servers too. On one site, Expect made the server forget that HEAD responses don't have a body and try to read too much data from the back-end socket, causing an upstream deadlock:

```http title:request
HEAD /<redacted> HTTP/1.1
Host: api.<redacted>
Content-Length: 6
Expect: 100-continue

ABCDEF
```

```HTTP title:response
HTTP/1.1 100 Continue

HTTP/1.1 504 Gateway Timeout
```

Other misbehaviours are less harmless, such as the multiple servers that respond to Expect by disclosing memory

```HTTP title:request
POST / HTTP/1.1
Host: <redacted>
Expect: 100-continue
Content-Length: 1

X
```

```HTTP title:response
HTTP/1.1 404 Not Found
HTTP/1.1 100 Continue

d

Ask the hotel which eHTTP/1.1 404 Not Found
HTTP/1.1 100 Continue

d
```

or secrets:
```http title:request
POST / HTTP/1.1
Host: <redacted>
Expect: 100-continue
Content-Length: 1

X
```

```http title:response
HTTP/1.1 401 Unauthorized
Www-Authenticate: Bearer
HTTP/1.1 100 ContinTransfer-EncodingzxWthTQmiI8fJ4oj9fzE"
X-: chunked

HTTP/1.1 401 Unauthorized
Www-Authenticate: Bearer
HTTP/1.1 100 ContinTransfer-EncodingzxWthTQm145
```

### Bypassing response header removal

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-13.png)

### 0.CL desync via vanilla Expect - T-Mobile

Simply sending a valid Expect header causes a 0.CL desync on numerous different servers.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-14.png)

### 0.CL desync via obfuscated Expect - Gitlab

Sending a lightly obfuscated Expect header exposes a substantial number of new targets. For example, `Expect: y 100-continue` causes a 0.CL desync on h1.sec.gitlab.net.

The site had a tiny attack surface so we weren't able to find a classic redirect or [Cross-Site Scripting (XSS)](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md) desync gadget for exploitation. Instead, we opted to shoot for [Response queue poisoning](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Exploiting%20Advanced%20Request%20Smuggling.md#Response%20queue%20poisoning):

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-15-1.png)

### CL.0 desync via vanilla Expect - Netlify CDN

Proving that it can break servers in every possible way, Expect can also cause [CL.0](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md) desync vulnerabilities.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-16.png)

### CL.0 desync via obfuscated Expect - Akamai CDN

Unsurprisingly, obfuscating the Expect header revealed even more [CL.0](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/CL.0%20smuggling%20vulnerabilities.md) desync vulnerabilities.

![](attachments/James%20Kettle%20-%20HTTP1.1%20must%20die%20the%20desync%20endgame-17.png)