---
raindrop_id: 1528754360
raindrop_highlights:
  695fa6a1c46dd776528c6692: cc90549331bc3371377aad6ae23e3dbb
  695fa6a456813960da0dedae: 9acf169a2666a87d949b3ce8dc39ac68
  695fa6c990fe5df30c679e8a: c6220fac3cfe95a655dfddc177b99c8c
  695fa6d730d0f5c18427e9e5: a3398944ede4750f7bd91ee3a4c60d88
  695fa6e9951963e79480624b: 6c0967370865b34b0cdc9a0f3a985a42
  695fa702c96b8c2fd5b2a4ab: 5cee9ba8f0e99e88e5601162cf6d7fd3
  695fa7088e05f19b66747edc: 71ad5e5d2660401f38781ce98c8022ea
  695fa72050e373a50315cc1e: e4f1f1d7e8bfd03b29e4a49cef839e17
  695fa73cc96b8c2fd5b2afa8: b4d1d559dde14d7d5faa9f67109d44d7
  695fa753cd2090a1172c10cd: 4460dd99731bc1c27d1a72a0dd0f1de9
  695fa769c46dd776528c8e8c: fbd6016f189ada6dc49c72490a508b5e
  695fa78bf4280c4afd072522: 68a53c348861d600242a5651d84cdc84
  695fa7b86d07df39d4e9c9b4: 8644fc3049bce3ffcc68587973315fb1
  695fa7c88e05f19b6674a2a3: d662e0edf5e20c45522e73aec0162612
  695fa7db8e05f19b6674a622: e8ca1cf39dee8503a52dc6e7d55fa0a6
  695fa7eab94e8de92f9a1b43: a9bb5d763a1bae2bf7ccb1feb88db967
  695fa7f650e373a50315f5ef: e81720ad6b43830b1ef0850da6933601
  695fa83e6d07df39d4e9e2d2: 7ad9321c1dd97700bc77187a2508bd90
  695fa86dc46dd776528cc033: 4593446dfa0a0b25d6bbb3585281717c
  695fa880f4280c4afd07522b: bbc69919f6c5e54644d55a197f3de282
  695fa8a7b94e8de92f9a3ff5: 3b9dc0277f0e40da6f34753e0ad03feb
  695fa90cc96b8c2fd5b30554: 5fb1ad1f4d0ff8ecb79ed886419a665b
title: "DNS rebinding attacks explained: The lookup is coming from inside the house!"

description: |-
  DNS rebinding attack without CORS against local network web applications. See how this can be used to exploit vulnerabilities in the real-world.

source: https://github.blog/security/application-security/dns-rebinding-attacks-explained-the-lookup-is-coming-from-inside-the-house/

created: 2026-01-06
sync-date: 1767878124246
tags:
  - "_index"

---
# DNS rebinding attacks explained: The lookup is coming from inside the house!

![](https://github.blog/wp-content/uploads/2024/02/Security-DarkMode-2-2.png)

> [!summary]
> DNS rebinding attack without CORS against local network web applications. See how this can be used to exploit vulnerabilities in the real-world.


DNS rebinding attack without [Cross-origin resource sharing (CORS)](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Cross-origin%20resource%20sharing%20(CORS).md) against local network web applications. We’ll walk you through the concept of [DNS rebinding](../Dev,%20ICT%20&%20Cybersec/Services/DNS%20-%20Domain%20Name%20System.md#DNS%20rebinding) from scratch, demystify how it works, and explore why it’s a serious browser-based security issue.

We’ll start by revisiting the [Same-origin policy (SOP)](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Same-origin%20policy%20(SOP).md), a fundamental part of web security, and show how DNS rebinding bypasses it. Attackers can use this technique to access internal applications running on your local machine or network, even if those apps aren’t meant to be publicly available.

>[!tldr]
>Attacks look like this:
>Here is the step-by-step attack:
>1.  You visit a website controlled by the hacker (e.g., `best-free-games.com`).
>2. Behind the scenes, the hacker owns the domain `best-free-games.com`. Initially, their DNS tells your computer: _"Hi! `best-free-games.com` is at IP address `1.2.3.4` (The Hacker's Server)."_
>3. Your browser loads the page. It’s just a normal website.
>4. The website has a hidden script running in the background. It waits for a moment and then asks the server for data again.
>5. **The Rebinding:** When your browser asks _"Where is `best-free-games.com` now?"_, the hacker's DNS server changes its answer. It now says: _"Oh, `best-free-games.com` is at IP address `127.0.0.1` (YOUR Computer)."_
>6. Your browser receives this new address. It thinks, _"Okay, I'm still talking to `best-free-games.com`, so I am allowed to talk to this IP."_
>	- It sends a request to `127.0.0.1` (which is actually your Deluge client).
>	- Because of the vulnerability, Deluge doesn't check credentials and blindly hands over your password file or configuration.  
>7. The script sends that password back to the hacker. Now they can log into your torrent client and potentially take over your computer.

## Same-origin policy

[Same-origin policy (SOP)](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Same-origin%20policy%20(SOP).md) is a cornerstone of browser security introduced in 1995 by Netscape. The idea behind it is simple: Scripts from webpages of one origin should not be able to access data from a webpage of another origin.

## The attack: DNS rebinding

People tend to think running something on localhost completely shields it from the external world. While they understand that they can access what is running on the local machine from their local browser, they miss that the browser may also become the gateway through which unsolicited visitors get access to the web applications on the same machine or local network.

>[!danger]
>If the resolved IP address of the webpage host changes, the browser doesn’t take it into account and treats the webpage as if its origin didn’t change. This can be abused by attackers.

For example, if an attacker owns the domain name `somesite.com` and delegates it to a DNS server that is under attacker control, they may initially respond to a DNS lookup with a public IP address, such as 172.217. 22.14, and then switch subsequent lookups to a local network IP address, such as 192.168.0.1 or 127.0.0.1.

Javascript loaded from the original `somesite.com` will run client-side in the browser, and all further requests from it to `somesite.com` will be **directed to the new, now local, IP address**.

From then on, documents loaded from different IP addresses—but resolved from the same hosts—will be considered to be of the same origin. This gives the attackers the ability to interact with the victim’s local network via Javascript running in the victim’s browser.

Other scenarios could include attackers abusing local VPN routes that are available to the targeted user, allowing access to corporate intranet web applications, for example.

### A real-world vulnerability

A real-world vulnerability was found in BitTorrent client Deluge (fixed in v2.2.0).

The Deluge BitTorrent client supports starting two services on system boot: daemon and WebUI.
We found a [Path Traversal](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Path%20Traversal.md) in an unauthenticated endpoint of the web application that allowed for arbitrary file read.

The `/js` endpoint of the WebUI component didn’t require authentication, since its purpose is to serve JavaScript files for the UI. The `request.lookup_path` was validated to start with a known keyword [1], but it could have been bypassed with `/js/known_keyword/../...`. The path traversal happened in [2], when the path was concatenated and later used to read a file [3].

A request to `/js/deluge-all%2F..%2F..%2F..%2F..%2F..%2F..%2F.config%2Fdeluge%2Fweb.conf`, for example, would return such information as the WebUI admin password SHA1 with salt and a list of sessions.

### Exploiting it

Even if the service is accessible only locally, since it is an unauthenticated endpoint, attackers could use a DNS rebinding attack to access the service from a specially crafted web site. 

For browsers that implement CORS-RFC1918, which segments address ranges into different address spaces (loopback, local network, and public network addresses), attackers could use a known Linux and MacOS bypass—the non-routable **0.0.0.0** IP address—**to access the local service**.

Let’s assume attackers know the port of the vulnerable application (8112 by default for Deluge WebUI), though discovering that the port can be automated with Singularity. A Deluge WebUI user opens a web page with multiple IFrames by visiting the malicious `somesite.com`. Each frame fetches http://sub.somesite.com:8112/attack.html. In order to bypass SOP, the **port number must be the same as the attacked application**. 

The DNS resolver the attackers control may respond alternately with 0.0.0.0, and the real IP address of the server with a very low time to live (TTL). When the DNS resolves with the real IP address, the browser fetches a page with a script that waits for the DNS entry to expire by checking if they can request and read http://sub.somesite.com:8182/js/deluge-all/..%2F..%2F..%2F..%2F..%2F..%2F.config%2Fdeluge%2Fweb.conf. If the attack succeeds, the script will have exfiltrated the configuration file.

For the full source of `attack.html` please check [this](https://securitylab.github.com/advisories/GHSL-2024-188_GHSL-2024-191_Deluge/) advisory.