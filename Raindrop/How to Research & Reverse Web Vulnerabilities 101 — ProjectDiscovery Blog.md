---
raindrop_id: 1489691188
raindrop_highlights:
  694000f861eb045264cd6b4e: 3782c14ce9e5375681038438aac29e9d
  6940010e8d03a1f3fcbd47d1: 1ce9a37515e61d8f94d34065160a1bff
  694001398d03a1f3fcbd50f6: c1929138ca86cbcd7fcfb71997275c8b
  6940013f066d3c4e6807a8f5: 2281e3821114ee45ad886afb84da5cc3
  69400153694a1f7082f7d80e: 889a17b31f46eabcdbfba59b6e3f1d6d
  6940015cb892135aa72786f7: 4e5276442ee08c6741245dfac314e126
  69400169b892135aa72788e7: 6803b65c44e90a0ee505e59c0e7ec657
  69400177519f79021cc9d44f: 41bb3ed4b2421d3160d697ae5c9021a4
title: "How to Research & Reverse Web Vulnerabilities 101 — ProjectDiscovery Blog"

description: |-
  Introduction
  
  This blog serves as a detailed methodology guide for analyzing, reversing, and researching web vulnerabilities, particularly those with CVEs assigned. The content outlines repeatable processes used to evaluate vague advisories, analyze vulnerable software, and ultimately recreate or validate security flaws. The objective is to establish a structured, replicable approach to web vulnerability research.
  
  
  Environment & Tools
  
  When approaching a new target for CVE research or reverse-e

source: https://projectdiscovery.io/blog/how-to-research-web-vulnerabilities

created: 1765788861660
type: link
tags: ["_index"]

 
  - "tech-blog" 
  - "Java" 
  - "PHP" 
  - "Tools" 
  - "dotNet"

---
# How to Research & Reverse Web Vulnerabilities 101 — ProjectDiscovery Blog

![](https://projectdiscovery.ghost.io/content/images/2025/11/How-to-Research---Reverse-Web-Vulns.png)

> [!summary]
> Introduction

This blog serves as a detailed methodology guide for analyzing, reversing, and researching web vulnerabilities, particularly those with CVEs assigned. The content outlines repeatable processes used to evaluate vague advisories, analyze vulnerable software, and ultimately recreate or validate security flaws. The objective is to establish a structured, replicable approach to web vulnerability research.


Environment & Tools

When approaching a new target for CVE research or reverse-e





Language-Specific Debugging Environments
For applications written in Java, remote debugging via the JVM is frequently leveraged. By enabling the -agentlib:jdwp flag during application startup, Java processes can be attached to a debugger such as IntelliJ IDEA Ultimate using the appropriate remote debugging configuration.
Identify where the Java application is started from, many times it would be using tomcat server and catalina.sh would be used, u can add flag such as  -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 to JAVA_OPTS environment variables and run the server again, this starts the debug server along with the application server.
Similarly, sometimes it would be straight forward java CLI that runs the application server, in that case as well you can run the command on your own add the same flag to start debug server.
Configuring Remote JVM Debug in IntelliJ IDEA
Add all relevant JAR files
To be able to debug Java application and their associated JAR files, in the IntelliJ IDEA, go to File > Project Structure > Libraries > click "+" and all the JAR files. Notice, these JAR files are now visible under "External libraries" in the left panel.
Configuring Debug Configuration
Click "Debug icon" -> add new configuration -> "Remote JVM debug" -> "Debug"
Set breakpoints on controllers, filters, deserializers, and template renderers.
Use Evaluate Expression on request objects to view parsed parameters, cookies, and headers.
.NET applications can be debugged effectively using tools like JetBrains dotPeek (for decompilation and dynamic analysis). With the proper symbol paths and local testbed, it’s possible to attach to running processes and debug both .NET Core and Framework-based services. For reverse-engineering scenarios where source is unavailable, dotPeek or ILSpy can reconstruct high-level code from binaries, enabling code-level reasoning about potential flaws.
For JavaScript/Node.js applications, VS Code with the official Node.js debugging extension is generally sufficient.
PHP applications can be instrumented using Xdebug with VS Code or PHPStorm.
Tooling Considerations
Decompilers
For Java applications, use IntelliJ IDEA for debuging and CFR, FernFlower, or Procyon for jar decompilation. IDEA will also automatically decompile jars for you.
For .NET decompiling and debugging, use ILSpy, dotPeek, or actively maintained dnSpyEx for interactive browse, search, and patch diff view.
Patch diffing
Git can be used for patch diffing if the source code is available
You can use VS code extensions as such as compare folders etc.
Even IntelliJ IDEA offers compare folder which also supports JAR decompiling on the fly.