# What is the HttpOnly flag in Cookies?

The _HttpOnly_ attribute is an optional attribute of the _Set-Cookie_ HTTP response header that prevents JavaScript code from accessing the specific cookie. It's usage is intended as a mitigation against [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) attacks.


# HttpOnly flag bypasses

## XSS + phpinfo

When a [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) and an [Information Disclosure](Information%20Disclosure.md) caused by a debug page such as `phpinfo` are present at the same time, the `HttpOnly` flag becomes useless because the cookies are contained, without protection, within the `phpinfo` page and can be extracted leveraging the XSS by issuing a fetch request to the page.

<iframe width="560" height="315" src="https://www.youtube.com/embed/tNFuy9h2pF0?si=j6B_kCAJ4gvelR8G&amp;start=188" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Cookie Eviction
![Cookie Eviction](Cookie%20Eviction.md)