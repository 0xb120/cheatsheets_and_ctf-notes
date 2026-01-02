---
author: zhero_web_security
aliases:
  - "Next.js and the Corrupt Middleware: The Authorizing Artifact"
tags:
  - readwise/articles
  - nextjs
url: https://zhero-web-sec.github.io/research-and-things/nextjs-and-the-corrupt-middleware?__readwiseLocation=
date: 2025-03-24
---
# Next.js and the Corrupt Middleware: The Authorizing Artifact

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)


## The Next.js middleware

> Middleware allows you to run code before a request is completed. Then, based on the incoming request, you can modify the response by rewriting, redirecting, modifying the request or response headers, or responding directly (*Next.js [documentation](https://nextjs.org/docs/app/building-your-application/routing/middleware)*). [](https://read.readwise.io/read/01jq477b56q7makw507r50genv)


Its use cases are numerous, but the most important ones include:
- Path rewriting
- Server-side redirects
- Adding elements such as headers (CSP, etc.) to the response
- And most importantly: Authentication and Authorization [](https://read.readwise.io/read/01jq478a7j15tzzc0vgj2pfsxg)

## The authorizing artifact: old code, 0ld treasure 

When a next.js application uses a middleware, the `runMiddleware` function is used [](https://read.readwise.io/read/01jq47ad5g0kbft6jf9pqsxmmf). It retrieves the value of the `x-middleware-subrequest` header and uses it **to know if the middleware should be applied or not**. [](https://read.readwise.io/read/01jq47apvs5kjtfje521bv7gp5)

The header value is split to create a list using the column character (`:`) as a separator and then checks if this list contains the `middlewareInfo.name` value. [](https://read.readwise.io/read/01jq47bzt059v65k35ghj0rkc7)

This means that if we add the `x-middleware-subrequest` header with the correct value to our request, the middleware - *whatever its purpose* - **will be completely ignored**, and the request will be forwarded via `NextResponse.next()` and will complete its journey to its original destination **without the middleware having any impact/influence on it**. [](https://read.readwise.io/read/01jq47cj0nm164cen045d8efaf)

But what is `middlewareInfo.name`? [](https://read.readwise.io/read/01jq47dfjktspt5vh0mp629gp9)

### Execution order and middlewareInfo.name

The value of `middlewareInfo.name` is perfectly guessable, it is only **the path in which the middleware is located**. [](https://read.readwise.io/read/01jq47e4qxth9fqdv7sxptf1rb)

Versions prior to 12.2 allowed nested routes to place one or more `_middleware` files anywhere in the tree. [](https://read.readwise.io/read/01jq47mdhwdtd2f8ym479m49e5)

**What does this mean for our exploit?**
> possibilities = numbers of levels in the path [](https://read.readwise.io/read/01jq47mn4rcrn90km28k3x6rtz)

So, to gain access to `/dashboard/panel/admin` (*protected by middleware*), there are three possibilities regarding the value of `middlewareInfo.name`, and therefore of `x-middleware-subrequest`:
`pages/_middleware`
or
`pages/dashboard/_middleware`
or
`pages/dashboard/panel/_middleware` [](https://read.readwise.io/read/01jq47ndze5ckans0reh1jhdgd)

>[!warning]
>**all versions of next.js** —*starting with version 11.1.4*— **were vulnerable..!** [](https://read.readwise.io/read/01jq47pebwsgts23p4nrp0gych)

Starting with version 12.2, the file no longer contains underscores and must simply be named `middleware.ts`. Furthermore, it must **no longer be located in the pages folder** [](https://read.readwise.io/read/01jq47qa4yf2hb3n9y1b5bnvk9)

With that in mind, the payload for the first versions starting with version 12.2 is very simple:
`x-middleware-subrequest: middleware` [](https://read.readwise.io/read/01jq47qj8j3cnwgaqb5n90wxq5)

It should also be taken into account that Next.js gives the possibility to create a `/src` directory. [](https://read.readwise.io/read/01jq47r669wpgzsvj06ps5gcxw)

In which case the payload would be:
`x-middleware-subrequest: src/middleware`

So, a total of **two possibilities, regardless of the number of levels in the path**. [](https://read.readwise.io/read/01jq47rekkb5snkwwk46sk5m7n)


### Max recursion depth
On more recent versions the logic has changed slightly again [](https://read.readwise.io/read/01jq47rv3a2phqn31f1ce2tr4t)

The value of the header `x-middleware-subrequest` is retrieved in order to form a list whose separator is the column character, as before. [](https://read.readwise.io/read/01jq47sec9aym3vpgaw7jrmysp)

This time, the condition for the request to be forwarded directly - *ignoring the rules of the middleware* - is [](https://read.readwise.io/read/01jq47sxvgm2jx767w8sg7rssj)

The value of the constant depth **must be greater or equal than the value of the constant** `MAX_RECURSION_DEPTH` (which is `5`), when assigned, the constant depth is incremented by 1 each time one of the values ​​of the list -`subrequests`- (being the result of the header value separated by `:`) is equal to the value `params.name` which is simply **the path to the middleware**. And as explained earlier, there are **only two possibilities**: `middleware` or `src/middleware`. [](https://read.readwise.io/read/01jq47th5k1xxfszc9sb4wrasp)

We just need to add the following header/value to our request in order to bypass the middleware:
`x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware`
or
`x-middleware-subrequest: src/middleware:src/middleware:src/middleware:src/middleware:src/middleware` [](https://read.readwise.io/read/01jq47tvrt6aa1d3ft9nc8j3b4)

## Authorization/Rewrite bypass
Here, when we try to access `/admin/login` we get a `404`. As we can see in the response header, a path rewrite is performed via the middleware to prevent unauthenticated/inappropriate users from accessing it:
![](https://zhero-web-sec.github.io/images/next-middleware-6.png)
But with our *authorizing artifact*:
![](https://zhero-web-sec.github.io/images/next-middleware-7.png)
We can access the endpoint without any problems [](https://read.readwise.io/read/01jq47w2ks54mbpb5x3cm5qxx4)

