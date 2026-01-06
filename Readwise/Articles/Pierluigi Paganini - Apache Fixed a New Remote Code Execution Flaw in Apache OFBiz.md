---
author: Pierluigi Paganini
aliases:
  - Apache Fixed a New Remote Code Execution Flaw in Apache OFBiz
tags:
  - readwise/articles
  - advisory
  - vulnerability-research/todo
  - Apache
  - Apache/OFBiz
url: https://securityaffairs.com/168106/security/apache-ofbiz-rce-cve-2024-45195.html
created: 2024-09-06
---
# Apache Fixed a New Remote Code Execution Flaw in Apache OFBiz

![rw-book-cover](https://securityaffairs.com/wp-content/uploads/2021/03/Apache-OFBiz.png)

## Highlights


> Apache fixed a high-severity vulnerability, tracked as [CVE-2024-45195](https://nvd.nist.gov/vuln/detail/CVE-2024-45195) (CVSS score: 7.5) affecting the Apache OFBiz open-source enterprise resource planning (ERP) system.
> [View Highlight](https://read.readwise.io/read/01j739q1g9qyy464kn1stha1ce)



> Apache OFBiz® is an open source product for the automation of enterprise processes that includes framework components and business applications.
> [View Highlight](https://read.readwise.io/read/01j739qa96g5e8awx920fqvr2d)



> The vulnerability is a Direct Request (‘Forced Browsing’) issue in Apache OFBiz. This flaw affects all versions of the software before 18.12.16.
>  *“Apache OFBiz below 18.12.16 is [vulnerable to](https://seclists.org/oss-sec/2024/q3/242) unauthenticated remote code execution on Linux and Windows. An attacker with no valid credentials can exploit missing view authorization checks in the web application to execute arbitrary code on the server.” reads the [analysis](https://www.rapid7.com/blog/post/2024/09/05/cve-2024-45195-apache-ofbiz-unauthenticated-remote-code-execution-fixed/) published by Rapid7. “Exploitation is facilitated by bypassing previous patches for [CVE-2024-32113](https://attackerkb.com/topics/3gj7c1dPY3/cve-2024-32113), [CVE-2024-36104](https://attackerkb.com/topics/IcT6K60n1c/cve-2024-36104), and [CVE-2024-38856](https://attackerkb.com/topics/W8Bc2zU52s/cve-2024-38856); this patch bypass vulnerability is tracked as [CVE-2024-45195](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-45195).”*
> [View Highlight](https://read.readwise.io/read/01j739r3j727z88pd5m0h0fjap)

