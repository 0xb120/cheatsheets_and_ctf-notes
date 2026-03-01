---
title: CSRFing Express with simple requests
source: https://blog.sparrrgh.me//web/2026/02/18/csrfing_qs.html
author:
  - Sparrrgh’s blog
published: 2026-02-18
created: 2026-02-28
description: Intro Have you ever had a perfectly good Cross-Site Request Forgery1 attack blocked by CORS? Cross-Site Request Forgery - Portswigger Web Academy ↩
tags:
  - clippings/articles
aliases:
  - Simple Request
  - Simple Requests
---
# CSRFing Express with simple requests

### Simple requests

A request won’t be pre-flighted if it’s a so called **[simple request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS#simple_requests)** (which basically means that it could be sent using a `<form>` tag).

To be considered as simple an HTTP request requires:

- Simple HTTP Methods
- Usage of CORS-safelisted headers only
- Simple Content-Types

Only GET, HEAD and POST are considered to be **simple methods**. This means that if you have an API authenticated via cookies which uses methods like PUT you won’t be able to attack that specific API.

The only **headers safelisted[^2]** when sending cross-origin requests are the following:

- Accept

- Accept-Language

- Content-Language

- Content-Type

- Range

Lastly, only three **content types** are allowed:

- application/x-www-form-urlencoded

- multipart/form-data

- text/plain

[^2]: [CORS safelisted headers - Fetch specification](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)