---
Description: Burp Suite is an integrated platform and graphical tool for **performing security testing of web applications**, it supports the entire testing process, from initial mapping and analysis of an application's attack surface, through to finding and exploiting security vulnerabilities.
URL: https://portswigger.net/burp
---

>[!tip] Reuse the same license on multiple devices
>1. Create a user with the same uid, gid and name of the original one (or use root)
>2. Copy the .jar file of burpsuite-pro
>3. Copy the `/root/.java/.userPrefs/burp/prefs.xml` file


# Useful extensions

>[!info]
>The ones marked with a \* are only available for the Pro Version

## Traffic Auditor

### *Retire.js

Integrates Burp with the Retire.js repository to find vulnerable JavaScript libraries. It passively looks at JavaScript files loaded and identifies those which are vulnerable.

![|500](../../zzz_res/attachments/retire-js.png)

### *Software Vulnerability Scanner

Scans for vulnerabilities in detected software versions using the Vulners.com API. It adds a new page where you can analyze all the findings and add your custom regex to inspect for, and it also adds issues to the target page.

![|500](../../zzz_res/attachments/software-vuln-scanner.png)

### *Software Version Reporter

Passively detects server software version numbers during scanning, spidering etc. It summarizes all the findings within a dedicated tab but it also generates issues within the Target tab.

![|500](../../zzz_res/attachments/software-version-reporter.png)
![|500](../../zzz_res/attachments/software-version-reporter2.png)

### *JS Miner

Find interesting stuff inside static files, like secrets, credentials, subdomains, API endpoint, etc. It **can be used both passively (default) or actively** (through the "Extensions" menu).

![|500](../../zzz_res/attachments/JS-miner.png)

### *Reflected Parameters

Monitors traffic and looks for request parameter values (longer than 3 characters) that are reflected in the response. **Launch active scans against those parameters** from the apposite tab.

![|700](../../zzz_res/attachments/reflected-parameters.png)

### *CSRF Scanner

Used to passively scan for [Cross-Site Request Forgery (CSRF)](../Web%20&%20Network%20Hacking/Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md#Cross-Site%20Request%20Forgery%20(CSRF)). A dedicated tab allows to customize token's names, etc. The scanner send requests passively in background.

![|800](../../zzz_res/attachments/CSRF-scanner.png)
![|500](../../zzz_res/attachments/CSRF-scanner2.png)

### *Error Message Checks

This extension passively reports detailed server error messages. You can configure arbitrary match results from the apposite tab.

![](../../zzz_res/attachments/error-message-checks.png)

### CSP Auditor

This extension provides a **readable view of CSP headers for responses**. It also **includes passive scan rules** to detect weak CSP configurations.

![|500](../../zzz_res/attachments/CSP-auditor.png)

### *CSP-Bypass

This extension is designed to **passively scan for CSP headers** that contain known bypasses as well as other potential weaknesses.

![|500](../../zzz_res/attachments/CSP-bypass.png)

### HTML5 Auditor

This extension checks for usage of HTML5 features that have potential security risks (client side storage, client geo-location, HTML5 client caches, web sockets)

![|500](../../zzz_res/attachments/html5-auditor.png)


## Active scanner enhancement

### *Active Scan++

ActiveScan++ extends Burp Suite's **active and passive scanning capabilities** with host header attacks, edge side includes, XML attacks, input transofmrations, blind code injection and CVEs. To invoke these checks, **just run a normal active scan**.

### *J2EEScan

Adds more than 80+ unique security test cases and new strategies to discover different kind of J2EE vulnerabilities.

### *Backslash Powered Scanning

This extension **complements Burp's active scanner** by using a novel approach capable of finding and confirming both known and unknown classes of server-side injection vulnerabilities. Evolved from classic manual techniques, this approach reaps many of the benefits of manual testing including casual WAF evasion, a tiny network footprint, and flexibility in the face of input filtering. Run the scan from the `Extension > Backslash Powered Scanner > diff-scan`

![|750](../../zzz_res/attachments/backslash-powered-scanner.png)
![](../../zzz_res/attachments/backslash-powered-scanner2.png)

### Java Deserialization Scanner

Gives Burp Suite the ability to find Java deserialization vulnerabilities. It **adds checks to both the active and passive scanner** and can also be used in an **"Intruder like" manual mode**, with a dedicated tab.

![|600](../../zzz_res/attachments/java-deserializarion-scanner.png)

### *Freddy, Deserialization Bug Finder

Helps with detecting and exploiting serialization libraries/APIs **with passive and active scans**.

![|750](../../zzz_res/attachments/freddy-deserialization.png)

### *NoSQL Scanner

This extension provides a way to discover NoSQL injection vulnerabilities. It **adds Passive and Active Scanner checks**.

### *CMS Scanner

**Active scan extension** for Burp that provides supplemental coverage when testing popular content management systems. 

### *NGINX Alias Traversal

Detects NGINX alias traversal due to misconfiguration. The extension implements an **active scanner check**. Simply run a new scan, preferably with an "Audit checks extensions only" configuration, on static resources identified via Burp's crawler.

![|900](../../zzz_res/attachments/nginx-alias-traversal.png)

### *IIS Tilde Enumeration Scanner

**Add an Active Scanner check** for detecting IIS Tilde Enumeration vulnerability and **add a new tab in the Burp UI to manually exploit the vulnerability**.

![|900](../../zzz_res/attachments/IIS-tilde-scanner.png)

## Fuzzing & Tiny scan

### *403 Bypasser

Tries different bypass techniques, including method switching, `..;` permutations, different headers, etc. 
The scan can be started from the `Extensions > 403 Bypasser` menu.

![](../../zzz_res/attachments/403-bypasser1.png)
![](../../zzz_res/attachments/403-bypasser2.png)

### Param Miner

Identifies hidden, unlinked parameters. It's particularly useful for finding web cache poisoning vulnerabilities.

![|900](../../zzz_res/attachments/param-miner.png)

### HTTP Request Smuggler

Supports scanning for Request Smuggling vulnerabilities, and also aids exploitation by handling cumbersome offset-tweaking for you.

![|900](../../zzz_res/attachments/request-smuggler.png)

### Server-Side Prototype Pollution Scanner

### *Collaborator Everywhere

Augments your in-scope proxy traffic by injecting non-invasive headers designed to reveal backend systems by causing pingbacks to Burp Collaborator.

### *Log4Shell Everywhere

Fork of James Kettle's excellent Collaborator Everywhere, with the injection parameters changed to payloads for the critical log4j CVE-2021-44228 vulnerability.

### *Upload Scanner

Implements most attacks that seem feasible for file uploads. The extension is testing various attacks and is divided into modules. Each module handles several attacks of the same category.

![|900](../../zzz_res/attachments/upload-scanner.png)

### *CORS, Additional CORS Checks

This extension can be used to test websites for CORS misconfigurations. It can spot trivial misconfigurations like arbitrary origin reflection, but also more sublte ones where a regex is not properly configured. "CORS* Additional CORS Checks" can be run in either **automatic** or **manual mode**.

![|900](../../zzz_res/attachments/additional-cors-checks.png)

### Command Injection Attacker

A comprehensive OS command injection payload generator. This extension is a customizable payload generator, suitable for detecting OS command injection flaws during dynamic testing.

![|900](../../zzz_res/attachments/shelling.png)

## API

### Wsdler

Takes a WSDL request, parses out the operations that are associated with the targeted web service, and generates SOAP requests that can then be sent to the SOAP endpoints.

![|900](../../zzz_res/attachments/wsdler.png)

### OpenAPI Parser

Aimed at streamlining the process of performing web service assessments involving OpenAPI based APIs.

![|900](../../zzz_res/attachments/openapi-parser.png)

## Authorization

### Autorize

Give to the extension the cookies of a low privileged user and navigate the website with a high privileged user. The extension automatically repeats every request with the session of the low privileged user and detects authorization vulnerabilities.

![|900](../../zzz_res/attachments/autorize.png)

### Auth Analyzer

Helps you to find authorization bugs. Just navigate through the web application with a high privileged user and let the Auth Analyzer repeat your requests for any defined non-privileged user. With the possibility to define Parameters the Auth Analyzer is able to extract and replace parameter values automatically.

![|900](../../zzz_res/attachments/auth-analyzer.png)

### AuthMatrix

Provides a simple way to test authorization in web applications and web services. Testers focus on thoroughly defining tables of users, roles, and requests for their specific target application upfront. These tables are displayed through the UI in a similar format to that of an access control matrix.

![|900](../../zzz_res/attachments/AuthMatrix.png)

## JSON and JWT analysis

### JSON Web Tokens

JWT4B lets you decode and manipulate JSON web tokens on the fly, check their validity and automate common attacks. Automatic recognition, JWT Editor, Resigning of JWTs, Signature checks, Automated attacks, Validity checks, Automatic tests for security flags in cookie transmitted JWTs.

![|800](../../zzz_res/attachments/json-web-tokens1.png)
![|800](../../zzz_res/attachments/json-web-tokens2.png)

### JWT Editor

Extension and standalone application for editing, signing, verifying, encrypting and decrypting JSON Web Tokens. Automatic detection and in-line editing of JWTs within HTTP requests/responses, signing and encrypting of tokens and automation of several well-known attacks against JWT implementations.

![JKWS_injection_1|800](../../zzz_res/attachments/JKWS_injection_1.png)
![JKWS_injection_2|800](../../zzz_res/attachments/JKWS_injection_2.png)
![JKWS_injection_3|800](../../zzz_res/attachments/JKWS_injection_3.png)

### JSON Decoder

Adds a new tab to Burp's HTTP message editor, and displays JSON messages in decoded form.

![|800](../../zzz_res/attachments/json-decoder.png)

### JSON Query

Parse and beautify JSON responses. Query JSON with JSONPath.

![|800](../../zzz_res/attachments/json-qeury.png)

### *GraphQL Raider

The gql query and variables are extracted from the unreadable json body and displayed in separate tabs. Not only the variables are extracted as insertion point for the scanner. Furthermore the values inside the query are also extracted as insertion point for the scanner.

![](../../zzz_res/attachments/graphql-rider1.png)
![](../../zzz_res/attachments/graphql-rider2.png)

### *InQL Introspection GraphQL Scanner

A full featured framework for enumerating and testing GraphQL.

![|900](../../zzz_res/attachments/inql1.png)



## Quality of life improvements and feature ehnancments

### Turbo Intruder

Extension for sending large numbers of HTTP requests and analyzing the results. It's intended to complement Burp Intruder by handling attacks that require extreme speed or complexity.

![Untitled|900](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%207.png)

### Content Type Converter

Converts data within JSON2XML, XML2JSON, HTTP2JSON. HTTP2XML

![](../../zzz_res/attachments/content-type-converter.png)

### .NET Beautifier

Beautifies .NET requests to make the body parameters more human readable. Built-in parameters like \_\_VIEWSTATE have their values masked. Form field names have the auto-generated part of their name removed.

### Hackvector

tag-based conversion tool that supports various escapes and encodings.

![](../../zzz_res/attachments/hackvector.png)

### Copy As Python-Requests

This extension copies selected request(s) as Python-Requests invocations.

### Request Timer

Captures response times for requests made by all Burp tools. It could be useful in uncovering potential timing attacks.

![|900](../../zzz_res/attachments/request-timer.png)

### Request Highlighter

Provides an automatic way to highlight HTTP requests based on headers content.

![](../../zzz_res/attachments/request-highlighter.png)

### Logger++

Multithreaded logging extension for Burp Suite. In addition to logging requests and responses from all Burp Suite tools, the extension allows advanced filters to be defined to highlight interesting entries or filter logs to only those which match the filter.

![](../../zzz_res/attachments/logger++.png)