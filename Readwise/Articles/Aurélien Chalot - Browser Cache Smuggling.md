---
author: Aurélien Chalot
aliases:
  - Browser Cache Smuggling
tags:
  - readwise/articles
url: https://blog.whiteflag.io/blog/browser-cache-smuggling/
created: 2024-08-20
---
# Browser Cache Smuggling

^135b64

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

## Highlights


> Therefore, moder browers implemented a mechanism that allow them storing these files locally on the computer in order not to reload them everytime time. This mechanism is called the browser cache.
> [View Highlight](https://read.readwise.io/read/01hcd32ke6039t4ydb7thm8s39)



> If we take a look at how Firefox works, on Windows, we’ll see that there is indeed a directory located in the localappdata directory that stores what looks like files
> [View Highlight](https://read.readwise.io/read/01hcd332mq37312hbyar3qprjj)



> browsers will not cache any files proposed by a server. It will cache static ressources
> [View Highlight](https://read.readwise.io/read/01hcd33kx56v6anbg9zdf2j2g0)



> our goal is to force the download of either a DLL or an executable. To do so, we’ll simply have to change the content type related to the dll and exe files from:
>  application/x-msdos-program com exe bat dll
>  To:
>  image/jpg com exe bat dll
> [View Highlight](https://read.readwise.io/read/01hcd34rwhpcpaxz1an3gxpsw0)



> when the DLL was downloaded and stored in the cache, it was renammed to a random filename without extension
> [View Highlight](https://read.readwise.io/read/01hcd3666t3b3t5gkb2nc87jvv)



> cached file is not just the DLL, it’s a specific file that contains both the file cached as well as metadata
> [View Highlight](https://read.readwise.io/read/01hcd37bfb7s6qvvgxm1tcg39q)



> we need to do it to create a flag in the HTTP response of the server that will allow us identifying our DLL
> [View Highlight](https://read.readwise.io/read/01hcd37x0yeewgb7ffbmj0kq11)



> # Adding the header used to find the real DLL location /calc.dll { add_header Tag DLLHERE; }
> [View Highlight](https://read.readwise.io/read/01hcd37ztypeay9e07tr1wj17c)



> using powershell or batch we can grep for this specific string in order to find our DLL in the local cache directory
> [View Highlight](https://read.readwise.io/read/01hcd38daw99q1whzx2z8sj2h5)

