---
author: zhero_web_security
aliases:
  - Next.js
  - Cache
  - and Chains The Stale Elixir
tags:
  - readwise/articles
  - nextjs
url: https://zhero-web-sec.github.io/research-and-things/nextjs-cache-and-chains-the-stale-elixir?__readwiseLocation=
date: 2025-03-05
---
# Next.js, Cache, and Chains: The Stale Elixir

Reading my previous publication is not required to understand this one, but if you’re tempted, here it is: [Next.js and Cache Poisoning: A Quest for the Black Hole](Vulnerability%20Researcher%20-%20Next.js%20and%20Cache%20Poisoning%20A%20Quest%20for%20the%20Black%20Hole.md)



## Data request

Before diving into the heart of the matter, it’s necessary to take a brief detour to understand the role of two Next.js functions that are crucial for what’s to come. These functions share an important commonality: they both transmit information to the target page. [](https://read.readwise.io/read/01jnk0k4ztwcenrwzt5gx79q5t)

- `getStaticProps` - **SSG (Static Site Generation)**:
 > If you export a function called getStaticProps (Static Site Generation) from a page, Next.js will pre-render this page at build time using the props returned by getStaticProps. ([@documentation](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props))
 
It’s quite straightforward: the function simply allows you to transmit data **already available during the build process** (*and therefore not tied to user requests*), which is, by its nature, meant to be **publicly cached**. [](https://read.readwise.io/read/01jnk0mfdk8mb59m7xyg5p6a34)

- `getServerSideProps` - **SSR (Server-Side Rendering)**:
 > getServerSideProps is a Next.js function that can be used to fetch data and render the contents of a page at request time. ([@documentation](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props))
 
Unlike `getStaticProps`, `getServerSideProps` transmits data that is only available at the time of requests, based on factors such as the user’s data who made the request: cookies, headers, URL parameters, etc. [](https://read.readwise.io/read/01jnk0n9n57sty3whf79cy7h6w)

For example, the following code retrieves the request’s user-agent and passes it to the page:
```js
 export async function getServerSideProps(context: GetServerSidePropsContext) {
	 const userAgent = context.req.headers['user-agent'];
	 return {
		 props: {
			 userAgent, 
		 },
	 };
}
```
 Finally, the data passed by `getServerSideProps` is in the form of a JSON object, as we will see shortly. [](https://read.readwise.io/read/01jnk0p6hdv86n0wvbjjjnm2pg)

### Data fetching

 When using either of these functions (whether for SSG or SSR), Next.js employs specific routes for data fetching. These routes follow this pattern: `/_next/data/{buildID}/targeted-page.json`
 - `buildID` is a unique identifier generated for each new build
 - `targeted-page` is the name of the page for which the data is retrieved

 The response is a JSON object named `pageProps` containing the transmitted data:
 ![](https://zhero-web-sec.github.io/images/p1.png) [](https://read.readwise.io/read/01jnk0rcr0sd5kvw2pnehxv9hf)


## Internal URL parameter, pageProps, and `__nextDataReq`

As I revisited the Next.js source code, I focused specifically on its internal operations, with the goal of finding ways to influence its behavior [](https://read.readwise.io/read/01jnk12vbdx2qt27eexmmw7t8e). I came across this particularly interesting [piece of code](https://github.com/vercel/next.js/blob/de47568e894e9c0b41312acfa916491142c03756/packages/next/src/server/base-server.ts#L2004):
 ![](https://zhero-web-sec.github.io/images/p2.png) *server/base-server.ts* [](https://read.readwise.io/read/01jnk13004syzvkncbc31824kq)

The name of the constant as well as the comments clearly indicate its purpose: its value is a boolean that determines **whether or not the request is a data request** (*as previously defined*). For the request to be classified as such:
 1. Either the URL parameter `__nextDataReq` must be present in the request **OR** the request must contain the header `x-nextjs-data` along with a specific server configuration
 2. It must be an SSG (Static Site Generation) request **OR** `hasServerProps` must return `true`, which is the case if `getStaticProps` or `getServerSideProps` is used on the page in question [](https://read.readwise.io/read/01jnk13qsqxrmryddk2n0hnvz1)

By sending a request to an endpoint that uses `getServerSideProps` and appending the `__nextDataReq` URL parameter, the server should return the expected JSON object instead of the HTML page: [](https://read.readwise.io/read/01jnk14612g060jg4er9a5d521)

![](https://zhero-web-sec.github.io/images/p3.png) [](https://read.readwise.io/read/01jnk14adqktttfjd7ravh3tde)


### Exploitation - DoS via Cache Poisoning

To trigger a DoS attack via cache poisoning by altering the content of different endpoints on the target site through the JSON object `pageProps`, the target site **must have a caching system**, and the URL parameters must **not be part of the cache-key**. 

This ensures that during the [content-negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation) phase, the caching system **doesn’t differentiate** between these two requests:
- A: `https://www.example.com`
 - B: `https://www.example.com/?__nextDataReq=1` [](https://read.readwise.io/read/01jnk1559hm8gzzjj2222zp1xx)

Consequently serving the response of request `B` -*cached by the attacker*- to users making request `A`. [](https://read.readwise.io/read/01jnk15mbfcca42pvzrcfz368d)

>[!warning]
>The content of the page will be altered, greatly impacting the user experience, which is obviously not desired by a bug bounty program. (*so be careful*)

 To address this, we can check if the `Accept-Encoding` header (*or another header*) is part of the cache-key. If it is, we can send the malicious request without the `Accept-Encoding` header, allowing us to check if the site is vulnerable without impacting it.
 Since **this header is automatically added by browsers**, “normal” users **will not** be served the response poisoned by the cache, [](https://read.readwise.io/read/01jnk18tk241hx3eamc2wg6rp1)

## CVE-2024-46982: The stale elixir (`x-now-route-matches`)

It all starts with this particularly [interesting conditional statement](https://github.com/vercel/next.js/blob/979fedb8d42b9e42f515e9e5451b5b3c96b97d53/packages/next/src/server/base-server.ts#L1991), which, when it returns true considers the request to be an SSG: [](https://read.readwise.io/read/01jnk1epapaj17ta3h8thj34zy)

![](https://zhero-web-sec.github.io/images/p5.png) *server/base-server.ts* [](https://read.readwise.io/read/01jnk1exxvjc34t3wxmny9ha80)

Since the data transmitted via `getServerSideProps` is **dynamic**, it is -*initially*[1]- not intended to be cached. [](https://read.readwise.io/read/01jnk1c5rkfee6ywy8bfm55gfx)


Therefore, it’s no surprise how the framework manages `Cache-Control`:
 - **Default SSR Cache-Control**: `Cache-Control: private, no-cache, no-store, max-age=0, must-revalidate`
 - **Default SSG Cache-Control**: `Cache-Control: s-maxage=31536000, stale-while-revalidate` [](https://read.readwise.io/read/01jnk1csx98bj28a0dk9q4czyt)

If we could have `true` in our first part of the conditional structure [](https://read.readwise.io/read/01jnk1ftk1ng72gv5spka05zq5), this would therefore allow a `Server Side Rendering` request to be passed off as a `Server Static Generation` request, **forcing its caching**. [](https://read.readwise.io/read/01jnk1g5y7n6twf9tcm5fhptsm)

It would therefore be sufficient for the header to be present in the request to achieve our goal. [](https://read.readwise.io/read/01jnk1gqrttdv98segy8sr0gkg)

![](https://zhero-web-sec.github.io/images/p6.png) [](https://read.readwise.io/read/01jnk1h3nxfykq1m3dm8ta5m61)

The `/poc` endpoint here uses the `getServerSideProps` function, so a **request containing SSR data**, which, as a reminder, contains **dynamic** data [](https://read.readwise.io/read/01jnk1hvjayv69j85fbg01jxnf). 

Now that the request is considered `SSG`, **caching is possible** as indicated by the new `Cache-Control: s-maxage=1, stale-while-revalidate` [](https://read.readwise.io/read/01jnk1kaa7xpx7v8mw239cs725)

- `s-maxage`: The `s-maxage` directive specifies how long a response **can be reused by a shared cache before it is considered stale** and requires a new request to the origin server. In our case, this duration is set to `1` second. [](https://read.readwise.io/read/01jnk1kfhwgmgppxy5ba6kczch)
- `stale-while-revalidate`:  `stale-while-revalidate` is a directive that tells the cache that **it can reuse a stale response while it revalidates one**. This means that once the end of the `max-age` is reached the cache is **allowed to use the stale response**. I specify that a duration can be specified to the `stale-while-revalidate` directive indicating the number of seconds during which the cache can use the stale response (*which is not the case here*). [](https://read.readwise.io/read/01jnk1m45sbneexbma3k6mkhx5)

### Exploitation - DoS via Cache Poisoning

By combining in the request:
 1. The internal URL parameter `__nextDataReq` to make it a **data request**
 2. The header `x-now-route-matches` to make it pass for an `SSG` thereby changing the `Cache-Control`

It is possible to cache the JSON object `pageProps` on the target endpoint **altering the content of any page**. [](https://read.readwise.io/read/01jnk1p7g4263td1ev5kzqypts)

Normal request to the `/poc` endpoint: ![](https://zhero-web-sec.github.io/images/p7.png) [](https://read.readwise.io/read/01jnk1q25ybbewewtw4sc1awyq)

Request to the `/poc` endpoint by adding the `__nextDataReq` parameter and the `x-now-route-matches` header: ![](https://zhero-web-sec.github.io/images/p9.png) [](https://read.readwise.io/read/01jnk1qbnnf2f8825ptd7ha3hh)

Poisoned page:
![](https://zhero-web-sec.github.io/images/p10.png)

### Exploitation - Stored XSS via Cache Poisoning

Since we’re now accessing the “normal” page without artificially modifying it, what about the `content-type`? It should no longer be a `application/json` response.. 

Looking at the poisoned response (`/poc`) in my proxy, when we directly access the poisoned endpoint (*without* `__nextDataReq`) the `content-type` is `text/html`! [](https://read.readwise.io/read/01jnk1s4e3zjab8wer1vzm8t0s)

Any value of the request (*initially sent by the attacker*) being reflected in the response is a vector for a [Stored Cross-Site Scripting (XSS)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md) [](https://read.readwise.io/read/01jnk1swe8anthnhcbpkhzy4f1). For an SXSS to be possible, **it only takes one element to be reflected**. [](https://read.readwise.io/read/01jnk1tna1qxr7x89tgf3y5djm)

```http
GET /poc?__nextDataReq=1 HTTP/1.1
Host: localhost:3000
User-Agent: CP TO SXSS ON NEXT.JS : <img src=x onerror=alert('Palestine')>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
x-now-route-matches: 1
```
[](https://read.readwise.io/read/01jnk1v4f09wcw4zpqbkwasnfj)

![](https://zhero-web-sec.github.io/images/p11.png) *Stored XSS on Next.js* [](https://read.readwise.io/read/01jnk1vd3rymp1e701pe9z2rrk)

>[!note]
>As explained earlier, when `getServerSideProps` is used, **it’s very likely** that an element from the request is reflected in the response, the main reason for this function being to transmit data only available at the time of the request. [](https://read.readwise.io/read/01jnk1w807s5j5dy05v11spw98)


### Exploitation - Another way

We saw that Next.js used specific routes for data fetching:
 > When using either of these functions (whether for SSG or SSR), Next.js employs specific routes for data fetching. These routes follow this pattern: `/_next/data/{buildID}/targeted-page.json` [](https://read.readwise.io/read/01jnk1xjt6nchg9jf3kvrq215r)

Note: *The* `buildId` *is returned by Next.js on pages within the script tags containing the id attribute* `__NEXT_DATA__`. [](https://read.readwise.io/read/01jnk1xwt2zvatp1rer86710qe)

Sending a request to the **data fetch route** by adding the `x-now-route-matches` header leads to poisoning the target page endpoint (`/poc`):
 ![](https://zhero-web-sec.github.io/images/p12.png)
 Response served by the poisoned cache when accessing `/poc`:
 ![](https://zhero-web-sec.github.io/images/p13.png)
 The result being exactly the same as with the use of the internal parameter, as the request is a **data request** in both cases. [](https://read.readwise.io/read/01jnk1zg3m0f30889d0835bkz1)


### Exploitation - Cache deception

 As you may have guessed, it is also possible to exploit the `stale-while-revalidate` aspect to perform a [Web Cache Deception](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Web%20Cache%20Deception.md) attack. [](https://read.readwise.io/read/01jnk20abvvnvz0z6nedm0gmwq)

