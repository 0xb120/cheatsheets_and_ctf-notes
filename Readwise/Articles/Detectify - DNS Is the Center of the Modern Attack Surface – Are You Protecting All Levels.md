---
author: Detectify
aliases:
  - DNS Is the Center of the Modern Attack Surface – Are You Protecting All Levels?
tags:
  - readwise/articles
url: https://blog.detectify.com/best-practices/dns-is-the-center-of-the-modern-attack-surface-are-you-protecting-all-levels/?__readwiseLocation=
created: 2025-03-21
---
# DNS Is the Center of the Modern Attack Surface – Are You Protecting All Levels?

![rw-book-cover](https://blogdetectify.cdn.triggerfish.cloud/uploads/2025/03/dns-center.png)


It’s common to configure exposure through the [DNS - Domain Name System](../../Dev,%20ICT%20&%20Cybersec/Services/DNS%20-%20Domain%20Name%20System.md). It’s not uncommon for an organization to have over 100,000 DNS records. [](https://read.readwise.io/read/01jpwear40b34v8h68bmnezpxv)

**DNS levels**
 A basic DNS lookup involves several key components that work together to complete the final query:
 1. **Root Servers:** There are 13 root servers worldwide, which serve as the authoritative source for DNS information. (Fun fact: the majority of these servers are located in NATO countries.)
 2. **Top-Level Domain (TLD):** These manage the different domain extensions, such as .com, .io, and others.
 3. **Registrars:** This is where you go to purchase your domain name.
 4. **DNS Providers:** Often associated with hosting services, content delivery networks (CDNs), or registrars, these providers facilitate DNS management.
 5. **Zones:** This refers to the configuration settings for the domain you have purchased.
 6. **Your software sends DNS queries:** This is the implementation of DNS that enables communication between machines that wish to connect with each other. [](https://read.readwise.io/read/01jpweb45x6e433524epc9exbc)

DNS can break at many levels – and it can be hard to detect [](https://read.readwise.io/read/01jpwebkgx3mvh75ne21bcgzg0)

**DNS lookup** 
 Establishing communication begins with a DNS lookup. [](https://read.readwise.io/read/01jpwecm9rqgkdfz8pzx6xnp4a)

![](https://blogdetectify.cdn.triggerfish.cloud/uploads/2025/03/DNS.png)
 • Step 1: Check your local DNS cache/stub-resolver. If the information is present, proceed to Step 9. If not, continue to Step 2.
 • Step 2: The resolver queries the root server for information about the Top-Level Domain (TLD).
 • Step 3: Retrieve information about the TLD and then proceed to Step 4.
 • Step 4: The resolver requests information from the TLD server about the specific domain.
 • Step 5: Obtain information about the domain and move on to Step 6.
 • Step 6: The resolver queries the domain for information regarding detectify.com.
 • Step 7: Retrieve information about detectify.com and then proceed to Step 8.
 • Step 8: The DNS resolver responds with the record information related to the domain.
 • Step 9: Connect to the server that operates the application for detectify.com.
 • Step 10: Enjoy state-of-the-art security testing! [](https://read.readwise.io/read/01jpwedz7ph5mpqvew5fjbshmc)

