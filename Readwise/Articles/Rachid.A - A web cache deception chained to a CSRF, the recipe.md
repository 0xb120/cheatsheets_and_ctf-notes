---
author: Rachid.A
aliases:
  - A web cache deception chained to a CSRF
  - the recipe
tags:
  - readwise/articles
url: https://infosecwriteups.com/a-web-cache-deception-chained-to-a-csrf-the-recipe-9e9a5b5f53aa
date: 2024-08-20
---
# A web cache deception chained to a CSRF, the recipe

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1116209/1xDM1fJfU_YV4Vl3MM_kP4w.jpeg)

## Highlights


> There are **several techniques** to *force* the caching of a response
> [View Highlight](https://read.readwise.io/read/01heabk7g1g4mvknkgj43b8z6j)



> if the extensions of these files (*css, js, png, etc.*) are filtered by the cache in order to “*adapt*” its behavior (saving the response or not), **adding these extensions following a URL containing sensitive information can result in a web cache deception**.
> [View Highlight](https://read.readwise.io/read/01heabkn33wa1rr5b9dft6v6vw)



> https://example.com/private_info.js?cachebuster=1 
>  https://example.com/private_info/.css?cachebuster=1 
>  https://example.com/private_info/;.png?cachebuster=1
> [View Highlight](https://read.readwise.io/read/01heabkpwk8aeds5ragns3x2v7)



> Once the attack vector has been found, simply send the link containing its **cache-buster(***) to the victim. When he clicks on the link, his personal information — *the nature of which will depend on the web app* — will be **accessible** to the attacker **via this same link**.
> [View Highlight](https://read.readwise.io/read/01heabmy8cqtmxswqbcqcpb8qt)

