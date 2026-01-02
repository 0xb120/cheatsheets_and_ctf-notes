---
raindrop_id: 1390410251
raindrop_highlights:
  68f77a08a6b978c854f968f8: 5a96c76d8495beea368e33c24795dd5e
  68f77a72320ea47e3dc16fcf: fba7b6f9b439257dee7511c450e79892
  68f77aaa9b2a1c0e2a10ccb8: 515fa1e344f4a50734aa981dc36c8934
  68f77aaea323f3a0dd9e7f55: 2414d79746950fb510b58d58c7a95202
  68f77ad11fcefb2637c442c8: 143a0ac8a3ba146b94d71b3cf1fc9b3a
  68f77ae02d6250e2970faa3d: e334ae8e35c054aafc1d6b7fb42adc8e
  68f77b43320ea47e3dc1abc5: 4865499789d726121c6455a5f6b9c623
  68f77b661de665f10209cf8f: 1fe3e4e8dba7f0c307f4d0d48c3c9e55
  68f77b79320ea47e3dc1b6ab: d910e2367244f839e3a33078b286790d
  68f77b908e676b99bf8c8b18: d4463e18cb7a4ba67d45cde48c46c6db
  68f77ba267f6795654d5419d: 4feb971dc7a1010dd6e376f0d1cddfe5
  68f77bec06401e380f4a3c8b: e54adde89fc89c8afc0a4d4a7da8c874
  68f77bfa1de665f10209f85f: d9b5b26ca7507b4c8951f7e054444f62
  68f77c682d6250e2971010e2: 11def9b737f38da7433ee736769ba365
  68f77c752d6250e297101318: 5430001b2aaacdfae1ec02f2527d70ff
  68f77ce3320ea47e3dc20b33: f13f237403796e424344608eeb8d6b6b
  68f77d24b241957228ef0de4: bd9fbeb74ea408265799e92423b3e7fb
  68f77d63b241957228ef1b08: 20afa970f8d6f78fc97a1707c82fcc5b
  68f77d821de665f1020a4517: 3f3e34df897cebe1df696ff13bb72eda
  68f77d9ea6b978c854fa4809: 0efb9a4d047981b87fdb5ee5ff20b9b5
  68f77dd5b241957228ef3634: 4be17c9e510d60bd0717fecacc1ba4a6
  68f77de662e7e5298fe5b4fb: 5c0c49d39a71e347889864c1890147b7
  68f77dfd1de665f1020a5fc1: 70f35f67eb91a53d1c221e1005aa337f
  68f77e078e676b99bf8d1585: 43502bc768926b9d4b0565f607f6a6c1
  68f77e24c7123731897fe392: fde54f0bcd8db5492454a04f589b4ff0
  68f77e8d89efd4de9d3911b5: 55ec5a23b31ba57c116dc5d458916390
  68f77ed771782e43095ccff1: 73f19b0225234691062b864efc24d9b0
  68f77f03b241957228ef72d2: 213959b204e0db06a759553153c39e9a
  68f77f231fcefb2637c53de8: 459e1d65ce367ada7e2bfad2967912e7
  68f77f49b241957228ef7fd7: bf503ecbf673cbb26220d2d071948de3
  68f77f5ca323f3a0dd9f8bc0: 62a4a47e76bfd6517ab4a8012f0e3265
title: "How to test NextJS applications"

description: |-
  Learn how to assess Next.js apps for SSRF, XSS, CSTI, SSTI, CSRF, cache issues, and data leaks. Practical tips, checks, and tools for bug bounty and pentesting.

source: https://deepstrike.io/blog/nextjs-security-testing-bug-bounty-guide

created: 1760511170695
type: link
tags: ["_index"]

 
  - "NextJS" 
  - "tech-blog"

---
# How to test NextJS applications

![](https://cdn.sanity.io/images/a3jopls3/testdataset/11eaea290c344685a8a2aadcb7084ce9f684261b-2144x1198.png)

> [!summary]
> Learn how to assess Next.js apps for SSRF, XSS, CSTI, SSTI, CSRF, cache issues, and data leaks. Practical tips, checks, and tools for bug bounty and pentesting.





Next.js Security Testing Guide for Bug Hunters and Pentesters
What is Next.js

Next.js is a full-stack React framework that supports server rendering and many server-side features out of the box. The JavaScript ecosystem can be split into libraries and frameworks
Library
A set of helper functions or components you call when you need them.
You control application flow directly, for example calling a function from jQuery.
Examples: Lodash, Axios, D3, jQuery.
Framework
Provides a larger structure and enforces conventions.
Often implements inversion of control: the framework calls your code at defined lifecycle points.
Examples: Angular, Nuxt (for Vue), Next.js (for React).
SSR (Server-Side Rendering): getServerSideProps runs on every request, generates HTML on the server, returns props for the React component that means it runs on every request from each user and generate a dynamic component and send it back to react to render the page on the browser
Static Site Generation (SSG)
getStaticProps builds HTML at build time. Good for pages that change rarely. The HTML is generated once during build and served as static content until the next build.
Incremental Static Regeneration (ISR)
SSG with a revalidate window. Pages are built at build time and revalidated periodically to rebuild updated static pages.
Middleware
Runs before requests reach pages or API routes. Useful for auth checks, redirects, rewrites, header modifications, and geo routing. Middleware runs on the server or edge and is not visible to the client.
API Routes
Placing files under /pages/api/* or app/api/* creates server-side endpoints that run on Node.js. Use these for server-only logic, database access, or background tasks without a separate Express server.
So, in practice you have two code locations:

Browser code: React components and client-side bundles that run in the user agent.
Server-only code: getServerSideProps, middleware, API routes, Server Actions, etc., which execute on the server and are not sent to the client.

What the browser receives are HTML responses and client-side JS bundles.
SSTI, CSTI, and XSS in Next.js applications
For SSTI:

Next.js SSR does not use traditional template engines, so SSTI is not a native risk. However, server side template injection can occur if the developer explicitly uses a template engine, EJS, Handlebars, or Pug, or evaluates untrusted input on the server, for example eval, Function(), or rendering with a template engine.
For CSTI and XSS:

As mentioned earlier, Next.js is built on React, and by default React escapes values before rendering them into the DOM. This means classic CSTI payloads, {{7*7}} or {{constructor.constructor('alert(1)')()}}, will not work in pure Next.js CSR. CSTI in Next.js CSR can happen only if unsafe coding patterns are used, such as the dangerouslySetInnerHTML function.
By default, React escapes dangerous characters, <, >, and &, so if userInput = "<script>alert(1)</script>", the browser will show it as text. Sometimes developers want to inject raw HTML directly into the DOM. For that, React provides a special prop.

<div dangerouslySetInnerHTML={{ __html: userInput }} />
Sensitive data leakage via __NEXT_DATA__ as SSR props leakage

getServerSideProps or other SSR logic can return sensitive data, API keys, internal URLs, or secrets, inside the page props. You can parse the page source for <script id="__NEXT_DATA__"> and inspect the JSON for secrets, tokens, or internal URLs.
Cache poisoning and cache deception

SSR or SSG pages, or user customized pages, may be cached by a CDN or a reverse proxy without appropriate Vary or Cache Control settings. Always monitor response cache headers like Vary and Cache Control to see whether you could cache another user’s sensitive data, cache deception, and visit it from another account to confirm whether you can see the same data.
Reverse front end JavaScript code to see the original files

In most frameworks, or even in plain JavaScript, developers like to minify code
When the bundler minifies the original multiple JavaScript files into one or more chunked files, it generates .map files. These files are used to reverse the minified files.

The link to these files is usually at the end of the JavaScript files as a comment like this.

//# sourceMappingURL=//app.example.com/some/path/file.js.map

If that line exists, the bundle references a map file, but the map file itself may be missing on the server. If the line is missing, the map was likely not generated or was stripped.
You can download this file by opening it directly and copying it into a new text file, or by using a curl command. Then put the JavaScript file and the .map file in the same directory and use any tool to reverse the minified file to its original state
SSRF in Next.js

Next.js has a component used to resize images so they can be cached or sent to the user to be rendered in multiple sizes. This function is enabled by default, and Next.js uses a URL like this.

https://domain.com/_next/image?url=/daoud.jpg&w=512&q=150
If the developer intentionally, or by accident, allows loading images from all websites by setting the hostname to *, this allows an attacker to request an image from any source and have it returned to the user.
Also, if dangerouslyAllowSVG is enabled, you can point to a malicious SVG file on your domain, which can lead to XSS.
Another attack vector for SSRF in Next.js is Server Actions.

What are Next.js Server Actions:

Server Actions are async functions that run on the server side but can be invoked from Client Components or Server Components. They are marked by the directive "use server" either at the top of the function or at the top of a module or file, which tells Next.js and React that this code is server only.
Next Action header

When a Client Component or <form action={someServerAction}> triggers a Server Action, Next.js does not always call a plain public URL. Instead, the client sends a POST to an internal Next endpoint and includes a header or form field that identifies the action.
That identifier appears as a header, commonly shown as Next-Action, and maps the incoming request to the specific server function to execute.
CSRF on Next.js applications:

Next.js is not immune by default to CSRF attacks, so developers must implement defense in depth, such as:

CSRF tokens in the request body rather than cookies.
An extra CSRF header.
The SameSite flag set to Strict, or at least Lax, on POST requests for session and important cookies.
Relying on an authorization header for authentication and authorization.
Dependency confusion on Next.js applications
Next.js as a framework is shipped via npm as packages, next, react, react-dom, and many transitive packages. The Next.js package itself depends on many other npm libraries for routing, bundling, server utilities, and more. If any of these public or private packages are deleted, or there is a typo in a package name, an attacker could upload a malicious package to the same registry with the same name. When the application updates dependencies, it may include the malicious package
If you are testing any Next.js application, add these files to your directory brute forcing wordlist.

package.json, package-lock.json, yarn.lock, .yarnrc.yml, pnpm-lock.yaml, pnpm-workspace.yaml, and .npmrc.

If any of these files are accessible to the public, you can download them and use this tool, https://github.com/visma-prodsec/confused, to quickly enumerate whether each package exists