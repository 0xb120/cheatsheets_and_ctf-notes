---
author: "sonarsource.com"
aliases: "Encoding Differentials: Why Charset Matters"
tags: RW_inbox, readwise/articles
url: 
date: 2025-02-04
---
# Encoding Differentials: Why Charset Matters

![rw-book-cover](https://assets-eu-01.kc-usercontent.com:443/b1ac63b6-1e65-01f4-6f38-e97c0e9214a1/37406917-8fc7-41c0-bfcd-03abe9bc8044/Charset-Matters_social-landscape.png)

## Highlights


If you have doubts about the `Content-Type` header, you are right. There is only a minor imperfection here: the header is **missing** a `charset` attribute.
[View Highlight](https://read.readwise.io/read/01j5xba0w3har0r7mcqzswd8ht)



Character Encodings[](https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters#character-encodings)
 A common `Content-Type` header in an HTTP response looks like this:
 HTTP/1.1 200 OK
 Server: Some Server
 Content-Type: text/html; charset=utf-8
 ...
 The `charset` attribute tells the browser that UTF-8 was used to encode the HTTP response body. A character encoding like UTF-8 defines a **mapping between characters and bytes**. When a web server serves an HTML document, it maps the characters of the document to the corresponding bytes and transmits these in the HTTP response body. This process turns characters into bytes (*encode*):
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='789'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/03789676-77cd-427e-90d8-d4451da66a27/encode.png?w=2024&h=789&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/03789676-77cd-427e-90d8-d4451da66a27/encode.png?w=2024&h=789&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 When the browser receives these bytes in the HTTP response body, it can translate them back to the characters of the HTML document. This process turns bytes into characters (*decode*):
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='761.0000000000001'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/6fe42fba-f2a0-4d0a-a461-aaf690ddcae3/decode.png?w=2024&h=761&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/6fe42fba-f2a0-4d0a-a461-aaf690ddcae3/decode.png?w=2024&h=761&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 UTF-8 is only one of **many character encodings** that a modern browser must support according to the [HTML spec](https://html.spec.whatwg.org/#character-encodings). There are plenty of others like `UTF-16`, `ISO-8859-xx`, `windows-125x`, `GBK`, `Big5`, etc.
[View Highlight](https://read.readwise.io/read/01j5xbas2wd3y1zn9hdqg85dxt)



A character encoding like UTF-8 defines a **mapping between characters and bytes**. When a web server serves an HTML document, it maps the characters of the document to the corresponding bytes and transmits these in the HTTP response body.
[View Highlight](https://read.readwise.io/read/01jk8t2ac7wgspfwzxzdrhmadm)



When the browser receives these bytes in the HTTP response body, it can translate them back to the characters of the HTML document.
[View Highlight](https://read.readwise.io/read/01jk8t2p81x9p7b0v4r8qjn12m)



But what if there is no `charset` attribute in the `Content-Type` header or it is invalid?
 In that case, the browser looks for a `<meta>` tag in the HTML document itself. This tag can also have a `charset` attribute that indicates the character encoding
[View Highlight](https://read.readwise.io/read/01j5xbbeb6ehb6a6pmrbhhbya6)



Another, less common way to indicate the character encoding is the [Byte-Order Mark](https://en.wikipedia.org/wiki/Byte_order_mark). This is a specific Unicode character (`U+FEFF`) that can be placed in front of a string to indicate the byte endianness and character encoding.
[View Highlight](https://read.readwise.io/read/01j5xbcgg9vv4f74t5qjw7518j)



A Byte-Order Mark at the beginning of an HTML document even takes precedence over a `charset` attribute in the `Content-Type` header
[View Highlight](https://read.readwise.io/read/01j5xbcvfcyypjvjapvnbdkafr)



In summary, there are three common ways that a browser uses to **determine the character encoding** of an HTML document, ordered by priority:
 1. Byte-Order Mark at the beginning of the HTML document
 2. `charset` attribute in the `Content-Type` header
 3. `<meta>` tag in the HTML document
[View Highlight](https://read.readwise.io/read/01j5xbd3pdcymcxfadjhg7nsms)



UTF-8 is only one of **many character encodings** that a modern browser must support according to the [HTML spec](https://html.spec.whatwg.org/#character-encodings). There are plenty of others like `UTF-16`, `ISO-8859-xx`, `windows-125x`, `GBK`, `Big5`, etc.
[View Highlight](https://read.readwise.io/read/01jk8t38y3cwv2tmfr4wgd14fn)



Similar to faulty HTML syntax, browsers try to recover from missing character set information when parsing the content served from a web server and **make the best of it**. This non-strict behavior contributes to a good user experience, but it may also **open doors for exploitation techniques** like [mXSS](https://www.sonarsource.com/blog/mxss-the-vulnerability-hiding-in-your-code/).
[View Highlight](https://read.readwise.io/read/01j5xbfcmvxfej8sj9j4zm67f4)



For missing character information, browsers try to make an educated guess based on the content, which is called [auto-detection](https://html.spec.whatwg.org/#encoding-sniffing-algorithm:~:text=The%20user%20agent%20may%20attempt%20to%20autodetect%20the%20character%20encoding%20from%20applying%20frequency%20analysis%20or%20other%20algorithms%20to%20the%20data%20stream.). This is similar to [MIME-type sniffing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types#mime_sniffing) but operates on a character encoding level.
[View Highlight](https://read.readwise.io/read/01j5xbg0we825xjqfyneas93hj)



The purpose of character encoding is to translate characters into a computer-processable byte sequence.
[View Highlight](https://read.readwise.io/read/01j5xbhpeaqvzxrnf09948g0tw)



If there is a **mismatch** between the character encoding used for encoding and decoding, the receiver may *see* different characters:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='747'%20width='1634'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/1b7f0442-bc33-4a34-94ab-494792bd0bd9/encode-decode-not-ok.png?w=1634&h=747&auto=format&fit=crop)
[View Highlight](https://read.readwise.io/read/01j5xbhxw7skq10jgy74r3rz5p)



For a web application, this becomes vital when user-controlled data is sanitized to prevent Cross-Site Scripting (XSS) vulnerabilities. If the character encoding that the browser assumes is different from what the web server intended, this could theoretically break the sanitization and lead to XSS vulnerabilities.
[View Highlight](https://read.readwise.io/read/01j5xbjpd6kxzzmpcqfdet4r6n)



even Google was prone to an issue like this [back in 2005](https://seclists.org/fulldisclosure/2005/Dec/att-1107/google_xss_211205.txt#:~:text=Google%27s%20404%20NOT%20FOUND%20mechanism). Google’s 404 page did not provide charset information, which could be exploited by inserting a [UTF-7](https://en.wikipedia.org/wiki/UTF-7) XSS payload. In UTF-7, HTML special characters like angle brackets are encoded differently from ASCII which can be leveraged to bypass sanitization:
 +ADw-script+AD4-alert(1)+ADw-+AC8-script+AD4-
[View Highlight](https://read.readwise.io/read/01j5xbkfgjg647d8847tvghnkh)



Although there are still a lot of other supported character encodings, most of these are not really useful from an attacker’s point of view. All **HTML special characters** like angle brackets and quotes are **ASCII only** and since most character encodings are ASCII-compatible, there is **no difference** for these characters. Even for UTF-16, which is not ASCII-compatible due to its fixed amount of two bytes per character, it is usually not possible to smuggle ASCII characters, because their corresponding byte representation is the same, just with a trailing (little-endian) or leading (big-endian) zero byte.
[View Highlight](https://read.readwise.io/read/01j5xbmxryp35yd1nncxcjd4cj)



Such a mismatch between the character encoding used for encoding and decoding is what we refer to as *Encoding Differential*
[View Highlight](https://read.readwise.io/read/01jk8tay2qm6799dez2t06mw0m)



However, there is a particularly interesting encoding: **ISO-2022-JP**.
[View Highlight](https://read.readwise.io/read/01j5xbn12npftnfq2kn8gn3xyd)



Particularly interesting about this encoding is that it supports certain **escape sequences** to **switch between different character sets**.
[View Highlight](https://read.readwise.io/read/01j5xbnfw7saed23tqvt3cqcxr)



For example, if a byte sequence contains the bytes `0x1b`, `0x28`, `0x42`, these bytes are not decoded to a character but instead indicate that all following bytes should be decoded using ASCII. In total, there are four different escape sequences that can be used to switch between the character sets ASCII, JIS X 0201 1976, JIS X 0208 1978 and JIS X 0208 1983:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='1013'%20width='1601'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
[View Highlight](https://read.readwise.io/read/01j5xbp2banh9ejbvc7knjcne4)



![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='1013'%20width='1601'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/7b209ce6-241c-4340-a6e6-7c097c6b9c5d/iso-2022-jp.png?w=1601&h=1013&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/7b209ce6-241c-4340-a6e6-7c097c6b9c5d/iso-2022-jp.png?w=1601&h=1013&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8tj8hwsvmr6whfm5p3yyaj)



there is another catch: at the time of writing, **Chrome (Blink) and Firefox (Gecko) auto-detect this encoding.** A single occurrence of one of these escape sequences is usually enough to convince the auto-detection algorithm that the HTTP response body is encoded with ISO-2022-JP.
[View Highlight](https://read.readwise.io/read/01j5xbpxw5hw7210c1b3nhm3wh)



two different exploitation techniques that attackers may use when they can make the browser assume an ISO-2022-JP charset. Depending on the capabilities of the attacker, this can for example be achieved by directly controlling the `charset` attribute in the `Content-Type` header or by inserting a `<meta>` tag via an HTML injection vulnerability. If a web server provides an invalid `charset` attribute or none at all, there are usually no other prerequisites since attackers can easily switch the charset to ISO-2022-JP via auto-detection.
[View Highlight](https://read.readwise.io/read/01j5xbqyth8ppvw095g0ph6nxv)



Technique 1: Negating Backslash Escaping[](https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters/#technique-1-negating-backslash-escaping)
 The scenario for this technique is that **user-controlled data** is placed **in a JavaScript string**:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='299'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/7b29a332-aebf-4789-b64a-9171ea0f6f38/iso-2022-jp-teq1-01.png?w=2024&h=299&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/7b29a332-aebf-4789-b64a-9171ea0f6f38/iso-2022-jp-teq1-01.png?w=2024&h=299&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8txk038y2397cn1bd5nspn)



Technique 1: Negating Backslash Escaping[](https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters#technique-1-negating-backslash-escaping)
 The scenario for this technique is that **user-controlled data** is placed **in a JavaScript string**:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='299'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/7b29a332-aebf-4789-b64a-9171ea0f6f38/iso-2022-jp-teq1-01.png?w=2024&h=299&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/7b29a332-aebf-4789-b64a-9171ea0f6f38/iso-2022-jp-teq1-01.png?w=2024&h=299&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Let’s imagine a website that accepts two query parameters called `search` and `lang`. The first parameter is reflected in a plaintext context and the second parameter (`lang`) is inserted into a JavaScript string:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='816'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/539746f8-646a-4ed5-8a85-b2e73a724284/iso-2022-jp-teq1-02.png?w=2024&h=816&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/539746f8-646a-4ed5-8a85-b2e73a724284/iso-2022-jp-teq1-02.png?w=2024&h=816&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 HTML special characters in the `search` parameter are HTML-encoded, and the `lang` parameter is properly sanitized by escaping double quotes (`"`) and backslashes (`\`). Thus, it is not possible to break out of the string context and inject JavaScript code:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='954'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/d0604dfd-fa98-46a7-b0cc-e6675d40744e/iso-2022-jp-teq1-03.png?w=2024&h=954&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/d0604dfd-fa98-46a7-b0cc-e6675d40744e/iso-2022-jp-teq1-03.png?w=2024&h=954&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The default mode for ISO-2022-JP is ASCII. This means that all bytes of the received HTTP response body are decoded with ASCII and the resulting HTML document looks like what we would expect:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='861.0000000000001'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/f439bfb5-f73d-4ef5-b217-62d0f9f027ea/iso-2022-jp-teq1-04.png?w=2024&h=861&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/f439bfb5-f73d-4ef5-b217-62d0f9f027ea/iso-2022-jp-teq1-04.png?w=2024&h=861&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Now, let’s assume an attacker inserts the escape sequence to switch to the JIS X 0201 1976 charset in the `search` parameter (`0x1b`, `0x28`, `0x4a`):
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='808'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/f2440d8e-b63f-4c61-9664-9cb6632f0049/iso-2022-jp-teq1-05.png?w=2024&h=808&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/f2440d8e-b63f-4c61-9664-9cb6632f0049/iso-2022-jp-teq1-05.png?w=2024&h=808&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The browser now decodes all bytes following this escape sequence with JIS X 0201 1976:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='977'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/0652ec9c-2f13-4196-9258-13450dd0dae6/iso-2022-jp-teq1-06.png?w=2024&h=977&auto=format&fit=crop)
[View Highlight](https://read.readwise.io/read/01j5xbv0s2c1262q17wj7shtv9)



Let’s imagine a website that accepts two query parameters called `search` and `lang`. The first parameter is reflected in a plaintext context and the second parameter (`lang`) is inserted into a JavaScript string:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='816'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/539746f8-646a-4ed5-8a85-b2e73a724284/iso-2022-jp-teq1-02.png?w=2024&h=816&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/539746f8-646a-4ed5-8a85-b2e73a724284/iso-2022-jp-teq1-02.png?w=2024&h=816&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8v0mr7vspp9xjy5j4n6b1a)



HTML special characters in the `search` parameter are HTML-encoded, and the `lang` parameter is properly sanitized by escaping double quotes (`"`) and backslashes (`\`).
[View Highlight](https://read.readwise.io/read/01jk8v1abbmv5q0wv3jn1j0rsb)



let’s assume an attacker inserts the escape sequence to switch to the JIS X 0201 1976 charset in the `search` parameter (`0x1b`, `0x28`, `0x4a`):
[View Highlight](https://read.readwise.io/read/01jk8v5e2jk82fwpgyt7kk50sk)



![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='808'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/f2440d8e-b63f-4c61-9664-9cb6632f0049/iso-2022-jp-teq1-05.png?w=2024&h=808&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/f2440d8e-b63f-4c61-9664-9cb6632f0049/iso-2022-jp-teq1-05.png?w=2024&h=808&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The browser now decodes all bytes following this escape sequence with JIS X 0201 1976:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='977'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
[View Highlight](https://read.readwise.io/read/01jk8v6jcjh4p5pvx05fxpmqdr)



this still results in the same characters as before, since JIS X 0201 1976 is *mainly* ASCII-compatible. However, if we closely inspect [its code table](https://en.wikipedia.org/wiki/JIS_X_0201#Codepage_layout), we can notice that there are two exceptions (highlighted in yellow):
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='1033'%20width='1104'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/05df20d0-2049-4f37-8414-7f956f34d5df/jisx-0201-codetable.png?w=1104&h=1033&auto=format&fit=crop)
[View Highlight](https://read.readwise.io/read/01jk8v71v30vaaegcaw9m61f1s)



As we can see, this still results in the same characters as before, since JIS X 0201 1976 is *mainly* ASCII-compatible. However, if we closely inspect [its code table](https://en.wikipedia.org/wiki/JIS_X_0201#Codepage_layout), we can notice that there are two exceptions (highlighted in yellow):
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='1033'%20width='1104'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/05df20d0-2049-4f37-8414-7f956f34d5df/jisx-0201-codetable.png?w=1104&h=1033&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/05df20d0-2049-4f37-8414-7f956f34d5df/jisx-0201-codetable.png?w=1104&h=1033&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The byte `0x5c` is mapped to the yen character (`¥`) and the byte `0x7e` to the overline character (`‾`). This is different from ASCII, where `0x5c` is mapped to the backslash character (`\`) and `0x7e` to the tilde character (`~`).
 This means that when the web server tries to escape a double quote in the `lang` parameter with a backslash, the browser does not *see* a backslash anymore, but instead a yen sign:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='978.9999999999999'%20width='2021'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/5afd4276-5177-4545-841d-7cdd3a9dc31c/iso-2022-jp-teq1-07.png?w=2021&h=979&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/5afd4276-5177-4545-841d-7cdd3a9dc31c/iso-2022-jp-teq1-07.png?w=2021&h=979&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Accordingly, the inserted double quote actually designates the end of the string and allows an attacker to inject arbitrary JavaScript code:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='563'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/bba1d004-96c5-4d59-a7a2-abf407e53184/iso-2022-jp-teq1-08.png?w=2024&h=563&auto=format&fit=crop)
[View Highlight](https://read.readwise.io/read/01j5xbxmx0veycksxxk6cf8y11)



The byte `0x5c` is mapped to the yen character (`¥`) and the byte `0x7e` to the overline character (`‾`). This is different from ASCII, where `0x5c` is mapped to the backslash character (`\`) and `0x7e` to the tilde character (`~`).
[View Highlight](https://read.readwise.io/read/01jk8v7zm1wagcw9rjhzq954sz)



when the web server tries to escape a double quote in the `lang` parameter with a backslash, the browser does not *see* a backslash anymore, but instead a yen sign:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='978.9999999999999'%20width='2021'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
[View Highlight](https://read.readwise.io/read/01jk8v8s9fa0yyn9esq96cxqvk)



Accordingly, the inserted double quote actually designates the end of the string and allows an attacker to inject arbitrary JavaScript code:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='563'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
[View Highlight](https://read.readwise.io/read/01jk8va108gpvydfxj1ss4vw3k)



Although this technique is quite powerful, it is limited to bypassing sanitization in a JavaScript context since a backslash character does not have special meaning in HTML.
[View Highlight](https://read.readwise.io/read/01jk8vb01k69an42jvrq6v6n6w)



Technique 2: Breaking HTML Context[](https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters/#technique-2-breaking-html-context)
 The scenario for this second technique is that an attacker can control values in **two different HTML contexts**. A common use case would be a website that supports markdown. For example, let’s consider the following markdown text:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='150'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/fb805311-69de-4451-a242-741b2fda8228/iso-2022-jp-teq2-01.png?w=2024&h=150&auto=format&fit=crop)
[View Highlight](https://read.readwise.io/read/01jk8vc80btv3rzv50jhy3evad)



Technique 2: Breaking HTML Context[](https://www.sonarsource.com/blog/encoding-differentials-why-charset-matters#technique-2-breaking-html-context)
 The scenario for this second technique is that an attacker can control values in **two different HTML contexts**. A common use case would be a website that supports markdown. For example, let’s consider the following markdown text:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='150'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/fb805311-69de-4451-a242-741b2fda8228/iso-2022-jp-teq2-01.png?w=2024&h=150&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/fb805311-69de-4451-a242-741b2fda8228/iso-2022-jp-teq2-01.png?w=2024&h=150&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The resulting HTML code looks like this:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='181'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/9466dd63-fe60-421e-9c5e-b069116cf47b/iso-2022-jp-teq2-02.png?w=2024&h=181&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/9466dd63-fe60-421e-9c5e-b069116cf47b/iso-2022-jp-teq2-02.png?w=2024&h=181&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Essential for this technique is that an attacker can control values in two different HTML contexts. In this case, these are:
 • Attribute context (image description/source)
 • Plaintext context (text surrounding images)
 By default, ISO-2022-JP is in ASCII mode and the browser *sees* the HTML document as expected:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='264'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/9d4c1bda-1e1e-4f10-be3d-1ac4ef7adaa7/iso-2022-jp-teq2-03.png?w=2024&h=264&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/9d4c1bda-1e1e-4f10-be3d-1ac4ef7adaa7/iso-2022-jp-teq2-03.png?w=2024&h=264&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Now, let’s assume an attacker inserts the escape sequence to switch the charset to JIS X 0208 1978 in the first image description:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='410'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/d64afc3a-7339-466a-ab2d-49e9c9f3fae1/iso-2022-jp-teq2-04.png?w=2024&h=410&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/d64afc3a-7339-466a-ab2d-49e9c9f3fae1/iso-2022-jp-teq2-04.png?w=2024&h=410&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 This makes the browser decode all bytes following with JIS X 0208 1978. This charset uses a fixed amount of 2 bytes per character and is not ASCII-compatible. This effectively breaks the HTML document:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='369'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/c6f350f6-d3c5-4e1c-abbb-b6f5bbfd01eb/iso-2022-jp-teq2-05.png?w=2024&h=369&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/c6f350f6-d3c5-4e1c-abbb-b6f5bbfd01eb/iso-2022-jp-teq2-05.png?w=2024&h=369&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 However, a second escape sequence can be inserted in the plaintext context between both images to switch the charset back to ASCII:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='423'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/63b0ca03-7816-472f-b096-c28a53a68703/iso-2022-jp-teq2-06.png?w=2024&h=423&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f3-758a-878c16f16a91/63b0ca03-7816-472f-b096-c28a53a68703/iso-2022-jp-teq2-06.png?w=2024&h=423&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 This way, all the following bytes are decoded using ASCII again:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='370'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/64ba5402-320b-01f
[View Highlight](https://read.readwise.io/read/01j5xc16fzx2qvbjfcf2y7ee7k)



The resulting HTML code looks like this:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='181'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/9466dd63-fe60-421e-9c5e-b069116cf47b/iso-2022-jp-teq2-02.png?w=2024&h=181&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/9466dd63-fe60-421e-9c5e-b069116cf47b/iso-2022-jp-teq2-02.png?w=2024&h=181&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8vcf28hecjka87a4jbbp6j)



Essential for this technique is that an attacker can control values in two different HTML contexts.
[View Highlight](https://read.readwise.io/read/01jk8vcrqztdbd4azfdzfg9xdf)



By default, ISO-2022-JP is in ASCII mode and the browser *sees* the HTML document as expected:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='264'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/9d4c1bda-1e1e-4f10-be3d-1ac4ef7adaa7/iso-2022-jp-teq2-03.png?w=2024&h=264&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/9d4c1bda-1e1e-4f10-be3d-1ac4ef7adaa7/iso-2022-jp-teq2-03.png?w=2024&h=264&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Now, let’s assume an attacker inserts the escape sequence to switch the charset to JIS X 0208 1978 in the first image description:
[View Highlight](https://read.readwise.io/read/01jk8vdgzqk43s5pa39jve6wrk)



This makes the browser decode all bytes following with JIS X 0208 1978. This charset uses a fixed amount of 2 bytes per character and is not ASCII-compatible. This effectively breaks the HTML document:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='369'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/c6f350f6-d3c5-4e1c-abbb-b6f5bbfd01eb/iso-2022-jp-teq2-05.png?w=2024&h=369&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/c6f350f6-d3c5-4e1c-abbb-b6f5bbfd01eb/iso-2022-jp-teq2-05.png?w=2024&h=369&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8vff4jgw11xwchg9gp8f2z)



However, a second escape sequence can be inserted in the plaintext context between both images to switch the charset back to ASCII:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='423'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/63b0ca03-7816-472f-b096-c28a53a68703/iso-2022-jp-teq2-06.png?w=2024&h=423&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/63b0ca03-7816-472f-b096-c28a53a68703/iso-2022-jp-teq2-06.png?w=2024&h=423&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8vg4yd3ytjegjt21963z6s)



This way, all the following bytes are decoded using ASCII again:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='370'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/3836339d-258a-48af-8d95-e4ee95e880ec/iso-2022-jp-teq2-07.png?w=2024&h=370&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/3836339d-258a-48af-8d95-e4ee95e880ec/iso-2022-jp-teq2-07.png?w=2024&h=370&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
[View Highlight](https://read.readwise.io/read/01jk8vgfgd92nvqsjgq4axn8eq)



we can notice that something changed. The beginning of the second `img` tag is now part of the `alt` attribute value:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='362'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/12f5839b-8242-4437-a569-b470641b5bb0/iso-2022-jp-teq2-08.png?w=2024&h=362&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/a8c9572d-fe62-0144-0642-b3f31f575091/12f5839b-8242-4437-a569-b470641b5bb0/iso-2022-jp-teq2-08.png?w=2024&h=362&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The reason for this is that the 4 bytes in between both escape sequences were decoded using JIS X 0208 1978. This also **consumed the closing double-quote** of the attribute value:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='668'%20width='2024'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
[View Highlight](https://read.readwise.io/read/01jk8vhzg6xdfgx1qz85r35rpf)

