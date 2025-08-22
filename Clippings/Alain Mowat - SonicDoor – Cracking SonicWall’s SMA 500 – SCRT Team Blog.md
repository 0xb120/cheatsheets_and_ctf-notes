---
title: SonicDoor – Cracking SonicWall’s SMA 500 – SCRT Team Blog
source: https://blog.scrt.ch/2025/06/04/sonicdoor-attacking-sonicwalls-sma-500/?__readwiseLocation=
author:
  - Author Alain Mowat
  - Alain Mowat
published: 2025-06-04
created: 2025-08-12
description: 
tags:
  - clippings/articles
---
# SonicDoor – Cracking SonicWall’s SMA 500 – SCRT Team Blog

> [!summary]
> This blog post details the discovery of multiple vulnerabilities in SonicWall's SMA 500 appliance. The researcher first gained a root shell on a virtual machine by manipulating its memory. This led to the discovery of an Apache path confusion vulnerability, allowing the download of sensitive files like a session database. Further investigation uncovered several memory corruption flaws, including heap and stack-based buffer overflows in CGI binaries, some of which were unauthenticated. An exploit chain is outlined, leveraging a crash log leak to bypass memory protections. The post also describes two Multi-Factor Authentication (MFA) bypasses: one due to predictable backup codes and another involving spoofing certificate validation. The vulnerabilities were reported to SonicWall, patched, and assigned multiple CVEs.

I was able to get my hands on a trial VM directly from [Sonicwall](https://www.mysonicwall.com/)‘s website

After booting up the device, a network scan showed that only ports 80 and 443 were open, while the console displayed a CLI, but no direct way of getting a shell.

## Getting a shell

My first objective, as in most of these projects was to **execute arbitrary commands on the device**, as this would allow to extract the file system and also debug the running appliance. 

In order to do this, I used an old trick one of my former colleagues showed me. It works something like this:

1. Identify commands in the CLI that likely call operating system commands
2. Pause the virtual machine
3. Search strings linked to the OS commands in the virtual machine’s saved memory file
4. Replace the expected command with another (such as bash) while maintaining the same memory size
5. Resume the virtual machine
6. Call the CLI command that has been “poisoned”

In this specific case, I targeted the “Restart SSL VPN Services” CLI command which should call `/usr/src/EasyAccess/bin/EasyAccessCtrl restart`. I replaced that exact string in the VM’s memory with `/////////////////////////////////////bin/bash`; resumed the VM, called the command and got my root shell.

![](https://blog.scrt.ch/wp-content/uploads/2025/06/image-1.png)

First goal achieved

## Path Confusion

I spent some time analysing its configuration and ended up with the following understanding of how it was set up.

![](https://blog.scrt.ch/wp-content/uploads/2025/06/image-3-1024x530.png)

Apache simplified overview

While looking through Apache’s configuration, I discovered that it was vulnerable to the [🍊 Orange Tsai - Confusion Attacks](../Readwise/Articles/🍊%20Orange%20Tsai%20-%20Confusion%20Attacks%20Exploiting%20Hidden%20Semantic%20Ambiguity%20in%20Apache%20HTTP%20Server!.md) issue which was presented by Orange Tsai at Blackhat last year.

```
RewriteRule ^/(.+)\.[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+[A-Za-z0-9]*-[0-9]+.*\.css$      /$1.css
```

Verifying whether the vulnerability is present can be done by issuing the two following requests and checking that the contents are identical.

- `https://TARGET/fileshare.10.1.2.13-72sv.css`
- `https://TARGET/fileshare.css%3f10.1.2.13-72sv.css`

![](https://blog.scrt.ch/wp-content/uploads/2025/06/image-11-1024x404.png)

Downloading the SQLite database

![](https://blog.scrt.ch/wp-content/uploads/2025/06/image-12-1024x495.png)

Recovering the Apache log file