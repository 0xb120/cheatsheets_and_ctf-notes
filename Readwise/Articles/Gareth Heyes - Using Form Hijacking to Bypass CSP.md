---
author: Gareth Heyes
aliases:
  - Using Form Hijacking to Bypass CSP
tags:
  - readwise/articles
url: https://portswigger.net/research/using-form-hijacking-to-bypass-csp
date: 2024-08-20
---
# Using Form Hijacking to Bypass CSP

![rw-book-cover](https://portswigger.net/cms/images/30/57/15d8-twittercard-twitter.png)

## Highlights


## What is form hijacking?
>  Form hijacking isn't really a widely known technique; the idea is you have a HTML injection vulnerability that is protected by [Content Security Policy (CSP)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Content%20Security%20Policy%20(CSP).md). Then you use the HTML injection to inject your own form action by using the `formaction` attribute or injecting your own form to send data to the attackers server. Over eager password managers will also help fill in credentials with injected input elements making the attack pretty serious.
> [View Highlight](https://read.readwise.io/read/01j0bdt696k75gew1ftxh9tjt6)



> We found a [real world example of this on Infosec Mastodon](https://portswigger.net/research/stealing-passwords-from-infosec-mastodon-without-bypassing-csp) where they used a fork of Mastodon that didn't filter HTML correctly. An attacker could then use form hijacking to send credentials to their server after Chrome's password manager had automatically filled them in. The end result was a user would see a post in Infosec Mastodon, click what looked like part of the interface but actually would send the user's credentials to an attacker's server.
> [View Highlight](https://read.readwise.io/read/01j0bdx1cgr3pmjgnadegv1x9x)



## Why does it happen?
>  The form-action directive was specified in version 2 of CSP. Unfortunately, default-src does not cover form actions. This means if you overlook this directive then your CSP will be vulnerable to form hijacking
> [View Highlight](https://read.readwise.io/read/01j0bdxbzd7588bvzyztnc71ck)



## In the wild

>  Whilst testing Burp we scanned our bug bounty pipeline and found lots of common mistakes that developers make when deploying CSP. We are going to highlight some of them below to help you avoid them.
>  Incorrect protocol
>  Some web sites forget the colon quite a lot when deploying.
>  `Content-Security-Policy: script-src 'self' https`
>  It should be:
>  `Content-Security-Policy: script-src 'self' https:`
>  You should avoid doing this of course because an attacker would be able to inject a script resource from any domain with TLS provided the target site is vulnerable to [XSS](https://portswigger.net/web-security/cross-site-scripting).
>  Missing semicolon
>  It's quite common to forget to include a semicolon. This can result in the directive name being used as a value which would mean the policy wouldn't enforce this directive!
>  This is incorrect:
>  `frame-ancestors 'self' https://example.com default-src 'none'`
>  It should be:
>  `frame-ancestors 'self' https://example.com; default-src 'none'`Incorrect value
>  In CSP all special directive values are quoted. It's quite common to see values not quoted and also illegal values like the following:
>  `Content-Security-Policy: frame-ancestors DENY`
>  There is no DENY value in the frame-ancestors directive value. It should be:
>  `Content-Security-Policy: frame-ancestors 'none'`None quoted hashes or nonces
>  A lot of sites also forget to include quotes around hashes or nonces. I think this is quite common because traditionally special values are quoted whereas non keywords are not. So it's quite understandable that they get confused:
>  This is incorrect:
>  `Content-Security-Policy: script-src sha512-BASE64HASH`
>  It should be:
>  `Content-Security-Policy: script-src 'sha512-BASE64HASH'`
> [View Highlight](https://read.readwise.io/read/01j0be1s1jhs5j3wh1tw3s46qk)

