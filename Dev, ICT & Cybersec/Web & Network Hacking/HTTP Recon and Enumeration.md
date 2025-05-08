See also [Passive information gathering (OSINT)](Passive%20information%20gathering%20(OSINT).md)
# High-level overview

Gathering information simply **browsing the site**. Look for:

- **Emails** and **Usernames**
- "**Contact us**" pages
- "**Social Media**" or "**Follow us**" pages and sections
- Interesting **notes** or **HTML comments**
- **Programming languages** and **frameworks** / **CMS** [^whatCMS]
- **URLs** and **Extensions**
- **Sitemaps**, **robots.txt** and **unnecessary exposed files**
- **Server OS**
- **Web server**
- **Server Headers**

[^whatCMS]: [What CMS Is This Site Using?](https://whatcms.org/)

## Proxy
- [Burpsuite](../Tools/Burpsuite.md)
- zap
- Caido

## Browser Plugins

- SwitchyOmega /FoxyProxy
- Cookie-Editor /CookieManager - Cookie Editor
- Wappalyzer
- User-Agent Switcher / User-Agent Switcher and Manager

## Other general tools

- [Out-of-Band Exfiltration Tools](../../Readwise/Articles/Piyush%20Kumawat%20(securitycipher)%20-%20Out-of-Band%20Exfiltration%20Tools.md)

---

# Technical Enumeration

## IP, Hosts, and Domains
### Direct IP Extraction

- **IP extraction by email**: If the web app uses some **web server's functions to send emails**, it will **expose the IP address** of the real server. Once we get the email from the web server (registration, newsletter, etc.) we will have to extract and examine the various **headers** and finally find the IP.

- **IP Extraction by Upload**: If the web app allows to **upload images and retrieve the URL from external resources**, it is possible use some [IP-logger](https://iplogger.org/) to get the real IP. Once the script has been uploaded in the form of an image, an HTTP request will be sent to our resource, which will immediately grab the information from the headers, including the IP.

### DNS Enumeration

- [Passive DNS Enumeration](Passive%20information%20gathering%20(OSINT).md#Passive%20DNS%20Enumeration)
- [DNS Enumeration](../Services/DNS%20-%20Domain%20Name%20System.md#Enumeration)

### VHOST Enumeration

- [Virtual host (VHost) enumeration](../../Readwise/Articles/blackbird-eu%20-%207%20Overlooked%20Recon%20Techniques%20to%20Find%20More%20Vulnerabilities.md#Virtual%20host%20(VHost)%20enumeration)


## Files, Directories, and Tech. Stack
### Endpoint Fuzzing

- [ffuf](../Tools/ffuf.md)
- [gobuster](../Tools/gobuster.md)
- [feroxbuster](https://github.com/epi052/feroxbuster)

>[!important]
>Use **good [Dictionaries](Password%20Attacks.md#Dictionary%20generation) ** (or better, [create your own!](../../Readwise/Articles/blackbird-eu%20-%20Creating%20Custom%20Wordlists%20for%20Bug%20Bounty%20Targets%20A%20Complete%20Guide.md))

### Endpoint Crawling

- [waybackurls](https://github.com/tomnomnom/waybackurls) - Fetch all the URLs that the Wayback Machine knows about for a domain
- [gau](https://github.com/lc/gau) - Fetch known URLs from AlienVault's Open Threat Exchange, the Wayback Machine, and Common Crawl. [^1]
- [waymore](https://github.com/xnl-h4ck3r/waymore) [^waymore-tweet]
- [urlfinder](https://github.com/projectdiscovery/urlfinder) [^urlfinder] - passively gathering URLs without active scanning
- [JSA](../Services/HTTP%20&%20HTTPS.md#^e44cdc) (`automation.sh`) - crawl endpoints from JS files
- [LinkFinder](../../Readwise/Articles/novasecio%20-%20Recon%20for%20Bug%20Bounty%208%20Essential%20Tools%20for%20Performing%20Effective%20Reconnaissance.md#LinkFinder)

[^waymore-tweet]: [@xnl_h4ck3r on Twitter - Tweets From XNL -Н4cĸ3r](../../Readwise/Tweets/@xnl_h4ck3r%20on%20Twitter%20-%20Tweets%20From%20XNL%20-Н4cĸ3r.md#^3d52ca)
[^urlfinder]: [Last Week in Security (LWiS) - 2024-12-02](../../Readwise/Articles/Erik%20-%20Last%20Week%20in%20Security%20(LWiS)%20-%202024-12-02.md#^e83851)

### Tech. Stack and Component Enumeration

- [whatweb](https://github.com/urbanadventurer/WhatWeb) & [whatweb.net](https://connect.ibsa-pharma.fr/)
- [Security Headers](https://securityheaders.com/) (online)
- [SSLLabs](https://www.ssllabs.com/ssltest/) (online)
- [webanalyze](../../Readwise/Articles/httpsgithub.comrverton%20-%20GitHub%20-%20RvertonWebanalyze%20Port%20of%20Wappalyzer%20(Uncovers%20Technologies%20Used%20on%20Websites)%20to%20Automate%20Mass%20Scanning..md)
- [HTTPX](../../Readwise/Articles/novasecio%20-%20Recon%20for%20Bug%20Bounty%208%20Essential%20Tools%20for%20Performing%20Effective%20Reconnaissance.md#HTTPX)

### CMS identification and enumeration

- [whatcms.org](https://whatcms.org/)
- [wpscan](https://github.com/wpscanteam/wpscan) & [WPProbe](../../Readwise/Articles/httpsgithub.comChocapikk%20-%20GitHub%20-%20ChocapikkWpprobe%20A%20Fast%20WordPress%20Plugin%20Enumeration%20Tool.md) ([Wordpress](../Dev,%20scripting%20&%20OS/Wordpress.md) specific)
- [magescan](https://github.com/steverobbins/magescan) (magento specific)

### Component Monitoring

- [JavaScript file monitoring](../../Readwise/Articles/blackbird-eu%20-%207%20Overlooked%20Recon%20Techniques%20to%20Find%20More%20Vulnerabilities.md#JavaScript%20file%20monitoring)

## Screenshotter

- [webcap](../../Readwise/Articles/Erik%20-%20Last%20Week%20in%20Security%20(LWiS)%20-%202025-03-03.md#^e13b68)

## Vulnerability and Misconfiguration scanners

- [nikto](../Tools/nikto.md)
- [nuclei](../Tools/nuclei.md)
- [Wapiti-Scanner/Wapiti: Web Vulnerability Scanner Written in Python3](../../Readwise/Articles/httpsgithub.comwapiti-scanner%20-%20Wapiti-ScannerWapiti%20Web%20Vulnerability%20Scanner%20Written%20in%20Python3.md)
- [MrmtwojApache-Vulnerability-Testing Apache HTTP Server Vulnerability Testing Tool](../../Readwise/Articles/httpsgithub.commrmtwoj%20-%20MrmtwojApache-Vulnerability-Testing%20Apache%20HTTP%20Server%20Vulnerability%20Testing%20Tool%20PoC%20for%20CVE-2024-38472%20,%20CVE-2024-39573%20,%20CVE-2024-38477%20,%20CVE-2024-38476%20,%20CVE-2024-38475%20,%20CVE-2024-38474%20,%20CVE-2024-38473%20,%20CVE-2023-38709.md) ( #apache specific )
- [misconfig mapper](../../Readwise/Articles/novasecio%20-%20Intigriti%20Bug%20Bytes%20219%20-%20December%202024.md#^4a91a1) - automate security misconfiguration detection on your list of targets
- [HExHTTP](../../Readwise/Tweets/@7h3h4ckv157%20on%20Twitter%20-%20Tweets%20From%207h3h4ckv157.md#^d53888)

## Secrets enumeration

- [Porch Pirate](https://github.com/mandconsultinggroup/porch-pirate) [^porch-pirate] - Porch Pirate is the most comprehensive Postman recon / OSINT client and framework that facilitates the automated discovery and exploitation of API endpoints and secrets committed to workspaces, collections, requests, users and teams.
- [gitrob](Passive%20information%20gathering%20(OSINT).md#^5a3ff0) and [gitleaks](Passive%20information%20gathering%20(OSINT).md#^88d33b) for GitHub secrets, sensitive files, etc.


[^porch-pirate]: [Plundering Postman With Porch Pirate](../../Readwise/Articles/Mand%20Consulting%20Group%20-%20Plundering%20Postman%20With%20Porch%20Pirate.md), mandconsulting.ca

---
# Automation and Frameworks

Tools:
- [recon-ng](../Tools/recon-ng.md) (bit outdated)
- [theHarvester](../Tools/theHarvester.md) - The tool gathers names, emails, IPs, subdomains, and URLs by using multiple public resources
- [Amass](../../Readwise/Articles/novasecio%20-%20Recon%20for%20Bug%20Bounty%208%20Essential%20Tools%20for%20Performing%20Effective%20Reconnaissance.md#Amass) - in-depth attack surface management open-source tool
- [Eyewitness](../../Readwise/Articles/novasecio%20-%20Recon%20for%20Bug%20Bounty%208%20Essential%20Tools%20for%20Performing%20Effective%20Reconnaissance.md#Eyewitness) - probing live hosts and screenshotting them but also perform basic technology fingerprinting
- https://trickest.com/ (service that allows to script e run multiple command sequences, sharing i/o etc.)

Online Services:
- [shodan](https://www.shodan.io/) (online)
	- [Complete Guide to Finding More Vulnerabilities With Shodan and Censys](../../Readwise/Articles/novasecio%20-%20Complete%20Guide%20to%20Finding%20More%20Vulnerabilities%20With%20Shodan%20and%20Censys.md)


[^1]: [GAU](../../Readwise/Articles/novasecio%20-%20Recon%20for%20Bug%20Bounty%208%20Essential%20Tools%20for%20Performing%20Effective%20Reconnaissance.md#GAU)
