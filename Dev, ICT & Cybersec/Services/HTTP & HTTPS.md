---
Ports:
  - 80, 443, 8080
Description: Is an application layer protocol in the Internet protocol suite model for distributed, collaborative, hypermedia information systems.
aliases:
  - Web Application Security
---


# Web Application Enumeration

Main checks:

- **Programming languages** and **frameworks** / **CMS** [^whatCMS]
- **URLs** and **Extensions**
- **Page contents** and **comments**
- **Sitemaps**, **robots.txt** and **unnecessary exposed files**
- **Web server**
- **Server Headers**
- **Administrative Consoles** (/manager/html, /phpmyadmin, etc.)
- **Database software**
- **Server OS**

[^whatCMS]: [What CMS Is This Site Using?](https://whatcms.org/)

## Enumeration tools

### Directory fuzzer

- [fuff](../Tools/fuff.md)
- [gobuster](../Tools/gobuster.md)
- [feroxbuster](https://github.com/epi052/feroxbuster)
- dirsearch
- wfuzz
- **good wordlists**
	- [SecLists](https://github.com/danielmiessler/SecLists)
	- [Assetnote Wordlist](https://wordlists.assetnote.io/)
	- https://github.com/nice-registry/all-the-package-names (npm public package names)

### Proxy

- [Burpsuite](../Tools/Burpsuite.md)
- zap
- Caido

### Plugins and utilities

- SwitchyOmega (plugin)
- FoxyProxy (plugin)
- Cookie-Editor (plugin)
- CookieManager - Cookie Editor (plugin)
- Wappalyzer (plugin)
- User-Agent Switcher (plugin)
- User-Agent Switcher and Manager (plugin)

### CMS Enumeration & Scanner

- whatweb (tool)
- WhatCMS (online)
- CMSMap (tool)
- [nikto](../Tools/nikto.md) (scanner)
- wpscanner
- [unSharepoint](https://blog.cys4.com/tool/2020/12/21/unSharePoint) (scanner)
- [misconfig mapper](../../Readwise/Articles/novasecio%20-%20Intigriti%20Bug%20Bytes%20219%20-%20December%202024.md#^4a91a1) - automate security misconfiguration detection on your list of targets
- [MrmtwojApache-Vulnerability-Testing Apache HTTP Server Vulnerability Testing Tool](../../Readwise/Articles/httpsgithub.commrmtwoj%20-%20MrmtwojApache-Vulnerability-Testing%20Apache%20HTTP%20Server%20Vulnerability%20Testing%20Tool%20%20PoC%20for%20CVE-2024-38472%20,%20CVE-2024-39573%20,%20CVE-2024-38477%20,%20CVE-2024-38476%20,%20CVE-2024-38475%20,%20CVE-2024-38474%20,%20CVE-2024-38473%20,%20CVE-2023-38709.md)

### Other useful tools

- Collaborator / [cowitness](https://github.com/stolenusername/cowitness) (HTTP server and DNS server mimic)
- [ngrok](https://ngrok.com/)
- [nuclei](../Tools/nuclei.md)
- [Out-of-Band Exfiltration Tools](../../Readwise/Articles/Piyush%20Kumawat%20(securitycipher)%20-%20Out-of-Band%20Exfiltration%20Tools.md#^3bac3c)
- https://trickest.com/ (service that allows to script e run multiple command sequences, sharing i/o etc.)

## Source Code Recovery

There are various path you can try follow to retrieve an application source code:
- Get it from cloud marketplaces (AWS, Azure, GCP) and reverse it
- Get the docker image from [Dockerhub](https://hub.docker.com/)
- Contact sales or search for trial version
- Leak the source code using vulnerabilities ([File Inclusion (LFI & RFI)](../Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md), [XML External Entity Injection (XXE Injection)](../Web%20&%20Network%20Hacking/XML%20External%20Entity%20Injection%20(XXE%20Injection).md), RCE, etc.)
- Reverse Engineering
	- You can use [binwalk](https://github.com/ReFirmLabs/binwalk) or even better [unblob](https://unblob.org/)[^unblob] for extracting firmwares

[^unblob]: [Frycos Security Diary - Hacking Like Hollywood With Hard-Coded Secrets](../../Readwise/Articles/Frycos%20Security%20Diary%20-%20Hacking%20Like%20Hollywood%20With%20Hard-Coded%20Secrets.md)

Further information inside the language specific notes:
- [Java](../Dev,%20scripting%20&%20OS/Java.md)
- [dotNET](../Dev,%20scripting%20&%20OS/dotNET.md)

## White Box Analysis (Secure Code Review)

- If possible, always **enable database query logging** and **verbose errors**
- Use **debug print statements** in interpreted code
- Attempt to **live-debug** the target application ([dnSpy](../Tools/dnSpy.md) makes this relatively easy for .NET applications. The same can be achieved in the Eclipse IDE for Java applications although with a bit more effort)
	- For client-side languages: [67 Weird Debugging Tricks Your Browser Doesn't Want You to Know](../../Readwise/Articles/norbauer.com%20-%2067%20Weird%20Debugging%20Tricks%20Your%20Browser%20Doesn't%20Want%20You%20to%20Know.md)
- After checking **unauthenticated areas**, focus on **areas** of the application that are likely to receive less attention (i.e., authenticated portions of the application)
- **Investigate how sanitization of user input is performed**. Is it done using a trusted, opensource library, or is a custom solution in place?

### Source code analyses tools, rules and resources

>[!tip]
>Search for dangerous patterns or functions using regexes, both manually and with automated tools!

- [[semgrep]]
- [CodeQL](../Tools/CodeQL.md)
- [snyk.io](https://app.snyk.io/)
- [graudit](https://github.com/wireghoul/graudit) - grep rough audit - source code auditing tool
- [weggli](https://github.com/weggli-rs/weggli) and C/C++ ruleset from Marco Ivaldi [^ivaldi-weggli]
- [SonarSource](https://rules.sonarsource.com/) - 5000+ Static Analysis Rules across 30+ programming languages
- [cloc](https://github.com/AlDanial/cloc) - cloc counts blank lines, comment lines, and physical lines of source code in many programming languages.
- [The Web Application Hacker's Handbook](../../Personal/Book%20list/The%20Web%20Application%20Hacker's%20Handbook%20-%20Dafydd%20Stuttard%20Marcus%20Pinto.md) and [OWASP Code Review Guide v2](../../Personal/Book%20list/OWASP%20Code%20Review%20Guide%20v2.md) - List of dangerous keywords and signatures for PHP, ASP.NET, Perl, JavaScript and MySQL
- [How to Find XSS (Cross-Site Scripting) Vulnerabilities in WordPress Plugins and Themes](../../Readwise/Articles/Alex%20Thomas%20-%20How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes.md#How%20to%20Find%20XSS%20(Cross-Site%20Scripting)%20Vulnerabilities%20in%20WordPress%20Plugins%20and%20Themes)

[^ivaldi-weggli]: [A Collection of Weggli Patterns for C/C++ Vulnerability Research](../../Readwise/Articles/Marco%20Ivaldi%20-%20A%20Collection%20of%20Weggli%20Patterns%20for%20CC++%20Vulnerability%20Research.md), Marco Ivaldi

For software updates and patches, focus on **diffing** [^patch][^patch-2][^patch-3] **older and newer version**:
- Read every detail contained inside the advisory in order to understand the kind of vulnerability and what/where to search inside the code
- Use the GitHub online editor or if you prefer, use [git and do patch diffing this way](../Tools/git.md#patch%20diffing)
- When analyzing and researching N-days, follows the [CVE North Stars](https://cve-north-stars.github.io/) (a method to kickstart vulnerability research by taking advantage of the CVE information freely available)

[^patch]: [AppSecSchool - How to Extract a Patch](../../Readwise/Articles/AppSecSchool%20-%20How%20to%20Extract%20a%20Patch.md)
[^patch-2]: [Blog on Shielder - Hunting for ~~Un~~authenticated N-Days in Asus Routers](../../Readwise/Articles/Blog%20on%20Shielder%20-%20Hunting%20for%20~~Un~~authenticated%20N-Days%20in%20Asus%20Routers.md#id696283770)
[^patch-3]: [WordPress CVE Reversing](https://patchstack.com/academy/wordpress/getting-started/cve-reversing/)
## Black Box Analysis

Black box pentesting [^black-box-pt] refers to a security test done by third party penetration testers. These external experts act like threat actors to check how safe a computer system is. 

- The tester doesn’t know anything about the system before starting.
- They only use information anyone can find online.
- They can’t see the system’s code or how it’s built inside.
- They try to find and use weak spots like a real hacker would.

[^black-box-pt]: [What is Black Box Penetration Testing?](https://blog.securelayer7.net/black-box-penetration-testing/), securelayer7.net

Focus on:
- Validation
- Sanitization
- Normalization
- [Web Services & APIs](../Web%20&%20Network%20Hacking/Web%20Services%20&%20APIs.md)
	- SOAP → WSDL (`?wsdl`)
	- REST → Swagger [^swagger-xss], OpenAPI, Postman, etc.

[^swagger-xss]: [Hacking Swagger-Ui - From XSS to Account Takeovers](../../Readwise/Articles/Dawid%20Moczadło%20-%20Hacking%20Swagger-Ui%20-%20From%20XSS%20to%20Account%20Takeovers.md)

Evading restrictions:
- [Evading Restrictions](../Web%20&%20Network%20Hacking/Evading%20Restrictions.md)
- [Fuzzing](../Reversing%20&%20Binary%20Exploitation/Fuzzing.md)

---

# Attacks and Vulnerabilities

## Server side vulnerabilities

- [Information Disclosure](../Web%20&%20Network%20Hacking/Information%20Disclosure.md)
- [Remote Code Execution (RCE)](../Web%20&%20Network%20Hacking/Remote%20Code%20Execution%20(RCE).md)
- [Command Injection](../Web%20&%20Network%20Hacking/Command%20Injection.md)
- [SQL Injection](../Web%20&%20Network%20Hacking/SQL%20Injection.md)
- [NoSQL Injection](../Web%20&%20Network%20Hacking/NoSQL%20Injection.md)
- [SQL Truncation](../Web%20&%20Network%20Hacking/SQL%20Truncation.md)
- [Path Traversal](../Web%20&%20Network%20Hacking/Path%20Traversal.md)
- [File Inclusion (LFI & RFI)](../Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md)
- [Authentication Attacks](../Web%20&%20Network%20Hacking/Authentication%20Attacks.md)
- [Password Reset Poisoning](../Web%20&%20Network%20Hacking/Password%20Reset%20Poisoning.md)
- [Access control vulnerabilities](../Web%20&%20Network%20Hacking/Access%20control%20vulnerabilities.md)
- [Execution After Redirect (EAR)](../Web%20&%20Network%20Hacking/Execution%20After%20Redirect%20(EAR).md)
- [Host Header attacks](../Web%20&%20Network%20Hacking/Host%20Header%20attacks.md)
- [Business logic vulnerabilities](../Web%20&%20Network%20Hacking/Business%20logic%20vulnerabilities.md)
- [HTTP Parameter Pollution (HPP)](../Web%20&%20Network%20Hacking/HTTP%20Parameter%20Pollution%20(HPP).md)
- [HTTP Verb Tampering](../Web%20&%20Network%20Hacking/HTTP%20Verb%20Tampering.md)
- [Insecure File Upload](../Web%20&%20Network%20Hacking/Insecure%20File%20Upload.md)
- [Server Side Request Forgery (SSRF)](../Web%20&%20Network%20Hacking/Server%20Side%20Request%20Forgery%20(SSRF).md)
- [XML External Entity Injection (XXE Injection)](../Web%20&%20Network%20Hacking/XML%20External%20Entity%20Injection%20(XXE%20Injection).md)
- [Extensible Stylesheet Language Transformations Injection (XSLT Injection)](../Web%20&%20Network%20Hacking/Extensible%20Stylesheet%20Language%20Transformations%20Injection%20(XSLT%20Injection).md)
- [Insecure Deserialization & Object Injection](../Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md)
- [Server Side Template Injection (SSTI)](../Web%20&%20Network%20Hacking/Server%20Side%20Template%20Injection%20(SSTI).md)
- [Server-Side Includes Injection (SSI)](../Web%20&%20Network%20Hacking/Server-Side%20Includes%20Injection%20(SSI).md)
- [Edge-Side Includes Injection (ESI)](../Web%20&%20Network%20Hacking/Edge-Side%20Includes%20Injection%20(ESI).md)
- [Web Cache Attacks](../Web%20&%20Network%20Hacking/Web%20Cache.md#Web%20Cache%20Attacks)
- [JWT Vulnerabilities](../Web%20&%20Network%20Hacking/JWT%20Vulnerabilities.md)
- [Server-side prototype pollution vuln](../Web%20&%20Network%20Hacking/Prototype%20Pollution.md#Server-side%20prototype%20pollution%20vuln)
- [Class Pollution](../Web%20&%20Network%20Hacking/Class%20Pollution.md)
- [OAuth 2.0](../Dev,%20scripting%20&%20OS/OAuth%202.0.md)
- [OpenID Connect](../Web%20&%20Network%20Hacking/OpenID%20Connect.md)
- [HTTP Request Smuggling](../Web%20&%20Network%20Hacking/HTTP%20Request%20Smuggling.md)
- [Server-side pause-based desync](../Web%20&%20Network%20Hacking/Client-side%20desync%20attacks.md#Server-side%20pause-based%20desync)
- [Type Juggling (aka type confusion)](../Web%20&%20Network%20Hacking/Type%20Juggling%20(aka%20type%20confusion).md)
- [Mass Assignment](../Web%20&%20Network%20Hacking/Mass%20Assignment.md)
- [GraphQL vulnerabilities](../Web%20&%20Network%20Hacking/GraphQL%20vulnerabilities.md)
- [Race Condition](../Web%20&%20Network%20Hacking/Race%20Condition.md)
- [ReDoS](../Web%20&%20Network%20Hacking/ReDoS.md)
- [Confusion Attacks](../Web%20&%20Network%20Hacking/Confusion%20Attacks.md)
## Client side vulnerabilities

- [HTML and CSS Injection](../Web%20&%20Network%20Hacking/HTML%20and%20CSS%20Injection.md)
- [Cross-Site Scripting (XSS)](../Web%20&%20Network%20Hacking/Cross-Site%20Scripting%20(XSS).md)
- [Cross-Site Script Inclusion (XSSI)](../Web%20&%20Network%20Hacking/Cross-Site%20Script%20Inclusion%20(XSSI).md)
- [Client-side template injection (CSTI)](../Web%20&%20Network%20Hacking/Client-side%20template%20injection%20(CSTI).md)
- [JSONP vulnerabilities](../Web%20&%20Network%20Hacking/JSONP%20vulnerabilities.md)
- [MIME sniffing](../Web%20&%20Network%20Hacking/MIME%20sniffing.md)
- [Header Fixation](../Web%20&%20Network%20Hacking/Header%20Fixation.md)
- [Session Attacks and Session Prediction](../Web%20&%20Network%20Hacking/Session%20Attacks%20and%20Session%20Prediction.md)
	- [Cross-Site Request Forgery (CSRF)](../Web%20&%20Network%20Hacking/Cross-Site%20Request%20Forgery%20(CSRF).md)
	- [Session Fixation](../Web%20&%20Network%20Hacking/Session%20Fixation.md)
- [Open Redirection](../Web%20&%20Network%20Hacking/Open%20Redirection.md)
- Cross-Origin based attacks
	- [CORS based attacks](../Web%20&%20Network%20Hacking/CORS%20based%20attacks.md)
	- [Cross-Origin Request Forgery (CORF)](../Web%20&%20Network%20Hacking/Cross-Origin%20Request%20Forgery%20(CORF).md) (must be in SameSite scenario)
	- [Cross-Application Cookie Exposure](../../Readwise/Articles/Anurag%20Mondal%20-%20From%20Informational%20to%20Critical%20Chaining%20&%20Elevating%20Web%20Vulnerabilities.md)
- [SameSite](../Web%20&%20Network%20Hacking/SameSite%20Cookie%20Attribute.md) based attacks
	- [Cross-Origin Request Forgery (CORF)](../Web%20&%20Network%20Hacking/Cross-Origin%20Request%20Forgery%20(CORF).md#Cross-Origin%20Request%20Forgery%20(CORF))
	- [Cross-Application Cookie Exposure](../../Readwise/Articles/Anurag%20Mondal%20-%20From%20Informational%20to%20Critical%20Chaining%20&%20Elevating%20Web%20Vulnerabilities.md)
	- [Cookie Tossing](../Web%20&%20Network%20Hacking/Cookie%20Tossing.md)
	- [Cookie Eviction (Cookie Jar Overflow)](../Web%20&%20Network%20Hacking/Cookie%20Eviction.md)
- [Clickjacking](../Web%20&%20Network%20Hacking/Clickjacking.md)
- [WebSockets](../Web%20&%20Network%20Hacking/WebSockets.md)
- [DOM-based vulnerabilities](../Web%20&%20Network%20Hacking/DOM-based%20vulnerabilities.md)
	- [Universal Code Execution by Chaining Messages in Browser Extensions](../../Readwise/Articles/Spaceraccoon's%20Blog%20-%20Universal%20Code%20Execution%20by%20Chaining%20Messages%20in%20Browser%20Extensions.md)
- [Client-side prototype pollution vuln](../Web%20&%20Network%20Hacking/Prototype%20Pollution.md#Client-side%20prototype%20pollution%20vuln)
- [Browser-powered request smuggling](../Web%20&%20Network%20Hacking/HTTP%20Request%20Smuggling.md#Browser-powered%20request%20smuggling)
- [Fetch Diversion](../Web%20&%20Network%20Hacking/Fetch%20Diversion.md)
- [ReDoS](../Web%20&%20Network%20Hacking/ReDoS.md)

## API-specific attacks

- [SOAPAction spoofing](../Web%20&%20Network%20Hacking/SOAPAction%20spoofing.md)
- [Mass Assignment](../Web%20&%20Network%20Hacking/Mass%20Assignment.md)
- [HTTP Parameter Pollution (HPP)](../Web%20&%20Network%20Hacking/HTTP%20Parameter%20Pollution%20(HPP).md)

---


# Security Implementations and Mitigations

- [Access control security models](../Web%20&%20Network%20Hacking/Access%20control%20security%20models.md)
- [Authentication](../Web%20&%20Network%20Hacking/Authentication.md)
- [Cross-origin resource sharing (CORS)](../Web%20&%20Network%20Hacking/Cross-origin%20resource%20sharing%20(CORS).md)
- [Same-origin policy (SOP)](../Web%20&%20Network%20Hacking/Same-origin%20policy%20(SOP).md)
- [Content Security Policy (CSP)](../Web%20&%20Network%20Hacking/Content%20Security%20Policy%20(CSP).md)
- [X-Frame-Options](../Web%20&%20Network%20Hacking/X-Frame-Options.md)
- [X-Content-Type-Options](../Web%20&%20Network%20Hacking/X-Content-Type-Options.md)
- [Defenses against CSRF attacks](../Web%20&%20Network%20Hacking/Cross-Site%20Request%20Forgery%20(CSRF).md#Defenses%20against%20CSRF%20attacks)
- [HttpOnly Cookie Attribute](../Web%20&%20Network%20Hacking/HttpOnly%20Cookie%20Attribute.md)
- [SameSite Cookie Attribute](../Web%20&%20Network%20Hacking/SameSite%20Cookie%20Attribute.md)
- [HTTP/2](../Web%20&%20Network%20Hacking/HTTP-2.md)
- [WAF](../Dev,%20scripting%20&%20OS/WAF.md)
- [Captcha](../Dev,%20scripting%20&%20OS/Captcha.md)
- [Character Encodings 101](../Dev,%20scripting%20&%20OS/Character%20Encodings%20101.md)