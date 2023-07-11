# AngularJS XSS

AngularJS is a popular JavaScript library, which scans the contents of HTML nodes containing the `ng-app` attribute (also known as an **AngularJS directive**). When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces. This technique is useful when angle brackets are being encoded.

```javascript
{{$on.constructor('alert(1)')()}}
```


# AngularJS sandbox

>[!question] What it AngularJS sandbox?
>The AngularJS sandbox is a mechanism that prevents access to potentially dangerous objects and properties, such as `window`, `document`, or `__proto__` in AngularJS template expressions.

Note: it was eventually removed from AngularJS in version 1.6. However, many legacy applications still use older versions of AngularJS and may be vulnerable as a result.

## How does it work?

The sandbox works by parsing an expression, rewriting the JavaScript, and then using various functions to test whether the rewritten code contains any dangerous objects.

>[!example]
>- The `ensureSafeObject()` function checks whether a given object references itself. This is one way to detect the `window` object, for example. 
>  
>- The `Function` constructor is detected in roughly the same way, by checking whether the constructor property references itself.
>- The `ensureSafeMemberName()` function checks each property access of the object and, if it contains dangerous properties such as `__proto__` or `__lookupGetter__`, the object will be blocked. 
>- The `ensureSafeFunction()`function prevents `call()`, `apply()`, `bind()`, or `constructor()` from being called.

## Escape the AngularJS sandbox

A sandbox escape involves tricking the sandbox into thinking the malicious expression is benign. The most well-known escape uses the modified `charAt()` function globally within an expression: `'a'.constructor.prototype.charAt=[].join`.

The attack works by overwriting the function using the `[].join` method, which causes the `charAt()` function to return all the characters sent to it, rather than a specific single character. Due to the logic of the `isIdent()` function in AngularJS, it compares what it thinks is a single character against multiple characters. As single characters are always less than multiple characters, the `isIdent()` function always returns true:

```javascript
isIdent = function(ch) {
    return ('a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || '_' === ch || ch === '$');
}
isIdent('x9=9a9l9e9r9t9(919)')
```

Once the `isIdent()` function is fooled, you can inject malicious JavaScript. For example, an expression such as `$eval('x=alert(1)')` would be allowed because AngularJS treats every character as an identifier. Note that we need to use AngularJS's `$eval()` function because overwriting the `charAt()` function will only take effect once the sandboxed code is executed.

Reference: 
- https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs
- https://portswigger.net/research/dom-based-angularjs-sandbox-escapes

![](https://www.youtube.com/watch?v=6pGEVDderN4&list=PLhixgUqwRTjwJTIkNopKuGLk3Pm9Ri1sF&index=3&ab_channel=LiveOverflow)

## Constructing an advanced AngularJS sandbox escape

You may encounter sites that are more restrictive with which characters they allow. A site may prevent you from using double or single quotes. In this situation, you need to use functions such as `String.fromCharCode()` to generate your characters. Although AngularJS prevents access to the `String` constructor within an expression, you can get round this by using the constructor property of a string instead. This obviously requires a string, so to construct an attack like this, you would need to find a way of creating a string without using single or double quotes.

In a standard sandbox escape, you would use `$eval()` to execute your JavaScript payload, but in the lab below, the `$eval()` function is undefined. Fortunately, we can use the `orderBy` filter instead. The typical syntax of an `orderBy` filter is as follows: 
```js
[123]|orderBy:'Some string'

//Poc:
https://YOUR-LAB-ID.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1

//breakdown
search=1&
toString().constructor.prototype.charAt%3d[].join;
[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

## AngularJS CSP bypass

[Content Security Policy (CSP)](../Web%20&%20Network%20Hacking/Content%20Security%20Policy%20(CSP).md) bypasses work in a similar way to standard sandbox escapes, but usually involve some HTML injection. When the CSP mode is active in AngularJS, it parses template expressions differently and avoids using the `Function` constructor.

Depending on the specific policy, the CSP will block JavaScript events. However, AngularJS defines its own events that can be used instead. When inside an event, AngularJS defines a special `$event` object, which simply references the browser event object. You can use this object to perform a CSP bypass. On Chrome, there is a special property on the `$event/event` object called `path`. This property contains an array of objects that causes the event to be executed. The last property is always the `window` object, which we can use to perform a sandbox escape. By passing this array to the `orderBy` filter, we can enumerate the array and use the last element (the `window` object) to execute a global function, such as `alert()`. The following code demonstrates this:

```html
<input autofocus ng-focus="$event.path|orderBy:'[].constructor.from([1],alert)'">

PoC:
<script> location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x'; </script>

<!-- breakdown -->
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x';
```

>[!note]
>Notice that the `from()` function is used, which allows you to convert an object to an array and call a given function (specified in the second argument) on every element of that array. In this case, we are calling the `alert()` function. We cannot call the function directly because the AngularJS sandbox would parse the code and detect that the `window` object is being used to call a function. Using the `from()` function instead effectively hides the `window` object from the sandbox, allowing us to inject malicious code.
>
>\- PortSwigger Research [created a CSP bypass using AngularJS in 56 characters using this technique](https://portswigger.net/research/angularjs-csp-bypass-in-56-characters).