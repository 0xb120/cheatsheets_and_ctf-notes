### Identify a suitable cache oracle

A **cache oracle is simply a page or endpoint that provides feedback about the cache's behavior**. This needs to be cacheable and must indicate in some way whether you received a cached response or a response directly from the server. (HTTP headers, observable changes to dynamic content, distinct resone times, etc.).

Ideally, the cache oracle will also reflect the entire URL and at least one query parameter in the response. This will make it easier to notice parsing discrepancies between the cache and the application, which will be useful for constructing different exploits later.

If you can identify that a specific third-party cache is being used, you can also consult the corresponding documentation. For example, Akamai-based websites may support the header `Pragma: akamai-x-get-cache-key`, which you can use to display the cache key in the response headers:

```http
GET /?param=1 HTTP/1.1
Host: innocent-website.com
Pragma: akamai-x-get-cache-key

HTTP/1.1 200 OK
X-Cache-Key: innocent-website.com/?param=1
```

### Probe key handling

The next step is to **investigate whether the cache performs any additional processing** of your input when generating the cache key. You are looking for an additional attack surface hidden within seemingly keyed components. Specifically look at any transformation that is taking place. Is anything being excluded from a keyed component when it is added to the cache key? Common examples are excluding specific query parameters, or even the entire query string, and removing the port from the `Host` header.

>[!example]
>Hypothetical cache oracle is the target website's home page. This automatically redirects users to a region-specific page. It uses the Host header to dynamically generate the Location header in the response.

```http
GET / HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 302 Moved Permanently
Location: https://vulnerable-website.com/en
Cache-Status: miss
```

To test whether the port is excluded from the cache key, we first need to request an arbitrary port and make sure that we receive a fresh response from the server that reflects this input:

```http
GET / HTTP/1.1
Host: vulnerable-website.com:1337

HTTP/1.1 302 Moved Permanently
Location: https://vulnerable-website.com:1337/en
Cache-Status: miss
```

Next, we'll send another request, but this time we won't specify a port:

```http
GET / HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 302 Moved Permanently
Location: https://vulnerable-website.com:1337/en
Cache-Status: hit
```

Although the `Host` header is keyed, the way it is transformed by the cache allows us to pass a payload into the application while still preserving a "normal" cache key that will be mapped to other users' requests. This kind of behavior is the key concept behind all of the exploits.

### Identify an exploitable gadget

The final step is to identify a suitable gadget that you can chain with this cache key flaw. This is an important skill because the severity of any [Web Cache Poisoning Attacks](Web%20Cache%20Poisoning.md#Web%20Cache%20Poisoning%20Attacks) is heavily dependent on the gadget you are able to exploit.

These gadgets will often be classic client-side vulnerabilities, such as [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) and [Open Redirection](Open%20Redirection.md). By combining these with [Web Cache Poisoning](Web%20Cache%20Poisoning.md), you can massively escalate the severity of these attacks, turning a reflected vulnerability into a stored one. Instead of having to induce a victim to visit a specially crafted URL, your payload will automatically be served to anybody who visits the ordinary, perfectly legitimate URL.