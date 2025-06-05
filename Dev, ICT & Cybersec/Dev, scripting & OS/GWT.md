---
aliases:
  - Google Web Toolkit
---
## GWT 101

Google Web Toolkit (GWT) is a development framework that allows Java developers to create single page applications (SPA) without the need to write any JavaScript or (for the most part) HTML.

### The client-side code

The front-end HTML / JavaScript of a GWT application is written in Java using the frameworks various UI API - which feels similar to Java Swing. The difference is, once you’re done, instead of it being compiled to bytecode, the front-end Java is compiled into multiple obfuscated JavaScript code permutations. These permutations are loaded by a bootstrap file which identifies the browser in use when the application is accessed.

During compilation, obfuscation of the client-side code permutation happens by default, which makes it a challenge to investigate or leverage from an assessment point of view. However, it is interesting nonetheless due to the way in which GWT’s communication protocol works. I will dive into that in a minute, but what you should keep in mind is that both the client and server (in GWT applications ) are always aware of exactly what objects, methods, and services exists within the application.

> If you can understand the client-side code, you can most likely enumerate and map out all of the application’s functionality.

### GWT-RPC

Due to the front-end being written in Java, its development naturally makes use of native Java objects. However, JavaScript is a completely different language. As such, the method used to pass data between the client and server needs a way to retain and transfer this information.

GWT makes use of a custom communication protocol called **GWT-RPC** which *serializes* Java objects for transmission between the client and the server. Unlike many other serialization technologies, the client and server in GWT-RPC communication are both fully aware of all objects that should be transmitted during any given request / response.

This is achieved using a service policy file, which stores a listing of all serializable objects and their IDs for a given service.

Example request:
```java
POST /olympian/authenticationService HTTP/1.1
Host: 127.0.0.1:8888
Content-Type: text/x-gwt-rpc; charset=utf-8
X-GWT-Permutation: 4D2DF0C23D1B58D2853828A450290F3F
X-GWT-Module-Base: http://127.0.0.1:8888/olympian/
Content-Length: 251

7|0|7|http://127.0.0.1:8888/olympian/|D48D4639E329B12508FBCA3BD0FC3780|com.ecorp.olympian.client.asyncService.AuthenticationService|login|java.lang.String/2004016611|bob|password123|1|2|3|4|2|5|5|6|7|
```

- POST /olympian/authenticationService -> The Service Path
- X-GWT-Permutation: 4D2DF0C23D1B58D2853828A450290F3F -> The “strong name” for the client-side code permutation
- 7 -> Version 7 of the serialization protocol
- 0 -> RPC Flag values (0/1/2)
- 7 -> The length of the “string table” that immediately follows
- [STRING TABLE] -> Seven pipe-delimited strings that will be referenced to build the RPC call
- 1 -> http://127.0.0.1:8888/olympian/ -> The base URL for the GWT module
- 2 -> D48D4639E329B12508FBCA3BD0FC3780 -> The “strong name” of the service’s policy file, which outlines what objects and types can be serialised
- 3 -> com.ecorp.olympian.client.asyncService.AuthenticationService -> The remote service interface
- 4 -> login -> The method being called
- 2 -> The number of parameters the method expects
- 5 -> java.lang.String/2004016611 -> The declared type of the first parameter
- 5 -> java.lang.String/2004016611 -> The declared type of the second parameter
- 6 -> bob -> The index in the string table of the value of the first parameter
- 7 -> password123-> The index in the string table of the value of the second parameter

See other examples inside [From Serialized to Shell: Auditing Google Web Toolkit](https://srcincite.io/blog/2017/04/27/from-serialized-to-shell-auditing-google-web-toolkit.html).
For further details and examples, you can read *The GWT-RPC Wire Protocol* [^1].

### GWT Deserialization

https://bishopfox.com/blog/gwt-unpatched-unauthenticated-java-deserialization-vulnerability

## Tools

- [GWTMap](https://github.com/FSecureLABS/GWTMap) [^2]
- [GWTab](https://github.com/thehackerish/GWTab) for [Burpsuite](../Tools/Burpsuite.md)
- [GWT Insertion Point](https://github.com/PortSwigger/gwt-insertion-points) for [Burpsuite](../Tools/Burpsuite.md)
- [GET-Penetration-Testing-Toolset](https://github.com/GDSSecurity/GWT-Penetration-Testing-Toolset)

[^1]: https://docs.google.com/document/d/1eG0YocsYYbNAtivkLtcaiEE5IOF5u4LUol8-LL0TIKU/edit?tab=t.0#heading=h.amx1ddpv5q4m

[^2]: https://labs.withsecure.com/publications/gwtmap-reverse-engineering-google-web-toolkit-applications
