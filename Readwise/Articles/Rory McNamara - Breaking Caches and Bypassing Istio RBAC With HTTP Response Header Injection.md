---
author: Rory McNamara
aliases:
  - Breaking Caches and Bypassing Istio RBAC With HTTP Response Header Injection
  - Override Cache behavior on NGINX using X-Accel-Expires
tags:
  - readwise/articles
url: https://snyk.io/blog/breaking-caches-bypassing-istio-rbac/?__readwiseLocation=
date: 2024-08-20
---
# Breaking Caches and Bypassing Istio RBAC With HTTP Response Header Injection

![rw-book-cover](https://res.cloudinary.com/snyk/image/upload/v1671633565/wordpress-sync/blog-feature-toolkit.jpg)

## Caching in NGINX
 When appropriately configured with a cache path, #NGINX will cache or not cache responses based on the HTTP response headers present. Specifically, the `Cache-Control` and `Expires` headers can be used to define what responses are cached and for how long.
> [View Highlight](https://read.readwise.io/read/01j1047r05j8br9p97jm4bnce1)

### X-Accel-Expires

NGINX supports the use of the custom `X-Accel-Expires` header, which, when present, can be used to completely override the behavior of both the `Cache-Control` and `Expires` headers.
> [View Highlight](https://read.readwise.io/read/01j104a01y54489w256sv08ena)

This header is stripped out of a response by NGINX, but we can observe its impact based on NGINX’s caching behavior. In the following example, we exploit a header injection vulnerability to include the `X-Accel-Expires` header and force the vulnerable page to be cached, contradictory to the instructions of the `Cache-Control` header:
![blog-http-response-cb7](https://snyk.io/_next/image?url=https%3A%2F%2Fres.cloudinary.com%2Fsnyk%2Fimage%2Fupload%2Fv1718896514%2Fblog-http-response-cb7.jpg&w=1240&q=75)
![blog-http-response-cb8](https://snyk.io/_next/image?url=https%3A%2F%2Fres.cloudinary.com%2Fsnyk%2Fimage%2Fupload%2Fv1718896514%2Fblog-http-response-cb8.jpg&w=1240&q=75)

The second response above shows a `Cache Status` of `HIT`, indicating that this response was retrieved from the cache, even though the `Cache-Control` and `Expires` headers are present and require that the response is not cached.
> [View Highlight](https://read.readwise.io/read/01j104at205em07hzfab7fwqgn)

### Notes on cache-keys

You may have noticed above that the user had to browse to *exactly* the same URL as the attacker had exploited, critically, including the `X-Accel-Expires` header injection. This is because, by default, NGINX will take the full request URI, including query arguments, and use that to key the cache for future lookups. This means that to retrieve the same response that the attacker primed, the victim’s request must result in the same cache key. In our application, the exploitation of the response header injection requires HTTP query parameters, which will end up in the cache key. However, if the vulnerable application can be exploited in a way that doesn’t modify the cache key (for example, headers being reflected in the response), this could result in a much wider impact on every user browsing to a standard page — without the need to use an attacker-supplied link.
> [View Highlight](https://read.readwise.io/read/01j104cjf2xn8n5qe59179r3tf)

