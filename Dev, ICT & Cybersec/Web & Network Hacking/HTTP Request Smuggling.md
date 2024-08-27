# Request Smuggling 101

>[!question] What is HTTP request smuggling?
> Is a technique for **interfering with the way a web site processes sequences of HTTP requests** that are received from one or more users.

![](zzz_res/attachments/Pasted%20image%2020230402154059.png)

 Most HTTP request smuggling vulnerabilities arise because the HTTP specification provides two different ways to specify where a request ends: 
 - the `Content-Length` header: it specifies the length of the message body in bytes
 - the `Transfer-Encoding` header: can be used to specify that the message body uses chunked encoding. This means that the message body contains one or more chunks of data. Each chunk consists of the chunk size in bytes (expressed in hexadecimal), followed by a newline, followed by the chunk contents. The message is terminated with a chunk of size zero.
```http
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b
q=smuggling
0
```

Since the HTTP specification provides two different methods for specifying the length of HTTP messages, it is possible for a single message to use both methods at once, such that they conflict with each other.

## Detecting HTTP request smuggling vulnerabilities

Request smuggling attacks involve placing both the `Content-Length` header and the `Transfer-Encoding` header into a single HTTP request and manipulating these so that the front-end and back-end servers process the request differently. The exact way in which this is done depends on the behavior of the two servers:

-   **[CL.TE](CL.TE%20smuggling%20vulnerabilities.md)**: the front-end server uses the `Content-Length` header and the back-end server uses the `Transfer-Encoding` header.
-   **[TE.CL](TE.CL%20smuggling%20vulnerabilities.md)**: the front-end server uses the `Transfer-Encoding` header and the back-end server uses the `Content-Length` header.
-   **[TE.TE](TE.TE%20smuggling%20vulnerabilities.md)**: the front-end and back-end servers both support the `Transfer-Encoding` header, but one of the servers can be induced not to process it by obfuscating the header in some way.

## Detecting HTTP/2 request smuggling vulnerabilities

>[!TLDR] Research: 
> [HTTP/2: The Sequel is Always Worse](https://portswigger.net/research/http2)

[HTTP/2](HTTP-2.md) messages are sent over the wire as a series of separate "frames". Each frame is preceded by an explicit length field, which tells the server exactly how many bytes to read in. Therefore, the length of the request is the sum of its frame lengths.

In theory, this mechanism means there is no opportunity for an attacker to introduce the ambiguity required for request smuggling, as long as the website uses HTTP/2 end to end. In the wild, however, this is often not the case due to the widespread but dangerous practice of **[HTTP/2 downgrading](HTTP-2%20downgrading.md)**.

-   **[H2.CL](H2.CL%20smuggling%20vulnerabilities.md)**: the front-end servers will simply reuse `Content-Length` header's value in the resulting HTTP/1 request instead of suing it's own value
-   **[H2.TE](H2.TE%20smuggling%20vulnerabilities.md)**: the back-end server do not strip the `Transfer-Encoding` header when downgrading requests to HTTP/1

---

# Browser-powered request smuggling

> [!tldr] Research:
> [Browser-Powered Desync Attacks: A New Frontier in HTTP Request Smuggling](https://portswigger.net/research/browser-powered-desync-attacks)

Browser-powered request smuggling attacks turn victim's web browser into a desync delivery platform, shifting the request smuggling frontier by **exposing single-server websites** and internal networks.

-   **[CL.0](CL.0%20smuggling%20vulnerabilities.md)**: Back-end servers can be persuaded to ignore the `Content-Length` header and ignore the body of incoming requests.
-   **[H2.0](H2.0%20smuggling%20vulnerabilities.md)**: same as CL.0 when the server performs [HTTP/2 downgrading](HTTP-2%20downgrading.md)
-   **[Client-side desync attacks](Client-side%20desync%20attacks.md)**: is an attack that makes the victim's web browser desynchronize its own connection to the vulnerable website.

---

# Exploiting HTTP & HTTP/2 Request Smuggling

- [Bypass front-end security controls](Exploiting%20HTTP%20Request%20Smuggling.md#Bypass%20front-end%20security%20controls)
- [Revealing front-end request rewriting](Exploiting%20HTTP%20Request%20Smuggling.md#Revealing%20front-end%20request%20rewriting)
- [Bypassing client authentication](Exploiting%20HTTP%20Request%20Smuggling.md#Bypassing%20client%20authentication)
- [Capturing other user' requests](Exploiting%20HTTP%20Request%20Smuggling.md#Capturing%20other%20user'%20requests)
- [Using request smuggling to exploit reflected XSS](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20exploit%20reflected%20XSS)
- [Using request smuggling to turn an on-site redirect into an open redirect](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20turn%20an%20on-site%20redirect%20into%20an%20open%20redirect)
- [Using request smuggling to perform web cache poisoning](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20perform%20web%20cache%20poisoning)
- [Using request smuggling to perform web cache deception](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20perform%20web%20cache%20deception)
- [Request smuggling via CRLF injection](Exploiting%20HTTP-2%20Request%20Smuggling.md#Request%20smuggling%20via%20CRLF%20injection)
- [Response queue poisoning](Exploiting%20HTTP%20Request%20Smuggling.md#Response%20queue%20poisoning)
- [HTTP request tunneling](Exploiting%20HTTP-2%20Request%20Smuggling.md#HTTP%20request%20tunneling)
- [Exploiting CL.0 & H2.0 vulnerabilities](Exploiting%20Browser-powered%20Request%20Smuggling.md#Exploiting%20CL.0%20&%20H2.0%20vulnerabilities)

# External researches

- [HTTP Desync Attacks: Request Smuggling Reborn, PortSwigger](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)
- https://portswigger.net/research/http2
- [Browser-Powered Desync Attacks: A New Frontier in HTTP Request Smuggling, PortSwigger](https://portswigger.net/research/browser-powered-desync-attacks)
- [Making desync attacks easy with TRACE](https://portswigger.net/research/trace-desync-attack), portswigger.net