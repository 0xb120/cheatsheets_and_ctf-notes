---
author: Pierluigi Paganini
aliases:
  - Attackers Used a Public ASP.NET Machine to Conduct ViewState Code Injection Attacks
tags:
  - readwise/articles
  - dotnet
url: https://securityaffairs.com/173956/hacking/abusing-asp-net-machine-to-deploy-malware.html?__readwiseLocation=
date: 2025-04-24
summary: Microsoft researchers found that attackers are using a public ASP.NET machine key to deploy Godzilla malware, which poses a significant security risk. They discovered over 3,000 publicly disclosed keys that can be exploited for ViewState code injection attacks, allowing remote code execution on servers. Microsoft advises securely generating keys and investigating affected servers to prevent exploitation.
---
# Attackers Used a Public ASP.NET Machine to Conduct ViewState Code Injection Attacks

![rw-book-cover](https://securityaffairs.com/wp-content/uploads/2025/02/image-12.png)

ViewState allows ASP.NET Web Forms to preserve page and control state between postbacks. ViewState data is stored in a hidden field on the page and is Base64 encoded. **The ASP.NET page framework uses machine keys to protect ViewState against tampering and information disclosure**. [](https://read.readwise.io/read/01jphhwqa1mcb3pwgt91ky9x3e)

