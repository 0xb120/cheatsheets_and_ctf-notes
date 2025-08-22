>[!tip]
>Java compiler is **javac** from Java JDK

Manage multiple java versions and SDK using [sdkman](https://sdkman.io/):
```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

sdk install java 11.0.21-tem
sdk use java 11.0.21-tem

sdk install java 8.0.392-tem  # Install Java 8
sdk use java 8.0.392-tem
```

# Java 101

- `*.java` = java source code
- `*.class` = compiled java code
- `*.jar` = Java Archive, is a compressed data archive used to distribute collections of Java classes
- `*.war` = Web Application Archive, is used to collect multiple JARs and static content, such as HTML, into a single archive
- `*.ear` = Enterprise Application Archive, contain multiple JARs and WARs to consolidate multiple web applications into a single file
- `*.do` extension in URL = used for URL mapping scheme in compiled Java code
- `*.jsp` = Java Server Pages

```bash
# Compile a Java source code
javac -source 1.8 -target 1.8 test.java

# Create a JAR archive
jar cmvf META-INF/MANIFEST.MF test.jar test.class

# Create a WAR file
jar -cvf ../webshell.war *

# Execute java program
java -jar test.jar
java test.class
```

See as an example:
- [Custom Java deserialization chain](../Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md#Custom%20Java%20deserialization%20chain)


Execute dynamic java code with `jshell` [^jshell]

[^jshell]: https://docs.oracle.com/javase/9/jshell/introduction-jshell.htm#JSHEL-GUID-630F27C8-1195-4989-9F6B2C51D46F52C8

```java
$ jshell
| Welcome to JShell -- Version 11.0.6
| For an introduction type: /help intro

jshell> import java.util.Random;
...
jshell> /exit
```



## J2EE syllabus

- **Enterprise Java Bean** (EJB): relatively heavyweight software component that encapsulates the logic of a specific business function within the application. EJBs are intended to take care of various technical challenges that application developers must address, such as transactional integrity.
- **Plain Old Java Object** (POJO): an ordinary Java object normally used to denote objects that are user-defined
- **Java Servlet** [^servlets]: object that resides on an application server and receives HTTP requests from clients and returns HTTP responses
- **Java web container**: platform or engine that provides a runtime environment for Java-based web applications (eg. #Tomcat, #JBoss, etc.)

[^servlets]: https://en.wikipedia.org/wiki/Jakarta_Servlet

# Working with Java

>[!tip] Information gathering and enumeration methodology from other researchers
>- [Security Code Audit - For Fun and Fails](https://frycos.github.io/vulns4free/2022/05/24/security-code-audit-fails.html), Frycos

- Easy copy for interesting files into a custom directory:
```bash
mkdir ALL_JARS
find . -iname '*.jar' -exec cp {} ALL_JARS \;
```

- Compact all JAR files into one single "huge" JAR [^jarjarbigs]
- Debug all the things using **Eclipse IDE for Enterprise Java and Web Developers** or **Eclipse IDE for Java Developers**
	- Search or setup a **JDWP** (Java Debug Wire Protocol) interface our Eclipse instance could talk to [^compact-jar]

[^compact-jar]: [Security Code Audit - For Fun and Fails](https://frycos.github.io/vulns4free/2022/05/24/security-code-audit-fails.html), frycos
[^jarjarbigs]: [jarjarbigs](https://github.com/mogwailabs/jarjarbigs), mogwailabs

## Servlet mappings

We can use [Process Explorer](../Tools/Sysinternals%20Suite.md#Process%20Explorer) (or other similar tools) to gain additional insight into the Java process we are targeting:
- Working path
- Command used to ran the process
- User running the process
- ...

When analyzing Java application it's also always good searching:
- the `WEB-INF` folder → which is the Java’s default configuration folder path
- `web.xml` → a deployment descriptor file which determine how URLs map to servlets, which URLs require authentication, and other information [^1][^2]
```xml
...
<servlet-mapping>
	<servlet-name>AMUserResourcesSyncServlet</servlet-name>
	<url-pattern>/servlet/AMUserResourcesSyncServlet</url-pattern>
</servlet-mapping>
...
<servlet>
	<servlet-name>AMUserResourcesSyncServlet</servlet-name>
	<servletclass>com.adventnet.appmanager.servlets.comm.AMUserResourcesSyncServlet</servletclass>
</servlet>
...
```
- `doGet`, `doPost`, `doPut`, `doDelete`, `doCopy`, `doOptions`, `doFilter` etc. implementations inside decompiled source code
- `request.getParameter`


## Black-box information and source-code gathering 

If you find a [Path Traversal](../Web%20&%20Network%20Hacking/Path%20Traversal.md) or a [File Inclusion (LFI & RFI)](../Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md) vulnerability in a Java application, look at the following files to extract as much information as possible (and eventually also the application source):
- `/proc/self/cmdline` to understand how the web server was executed and with which arguments (often contains useful full path)
- `/proc/self/environ` to exfiltrate every available environment variable (often contains full path, credential and other useful information)
- `/proc/self/map` to understand which JAR, WAR o EAR files have been imported and where they are located
- `/proc/self/fd/{1..999}` to exfiltrate `STDOUT`, `STDERR` and every file loaded by the application (including JAR files, WAR, etc.)
- Web server logs and configuration files (read the relative documentation or use some dedicated wordlist)

## Decompile Java

>[!tip] Related resources 
>See also [Android 101](../Mobile%20Hacking/Android%20101.md) and [Android Application Security](../Mobile%20Hacking/Android%20Application%20Security.md)

- [JD-GUI](../Tools/JD-GUI.md)
- [jadx](../Tools/jadx.md)
- [Bytecode Viewer](../Tools/Bytecode%20Viewer.md)

# Java tricks

- [Hiding Payloads in Java Source Code Strings](../../Readwise/Articles/PortSwigger%20Research%20-%20Hiding%20Payloads%20in%20Java%20Source%20Code%20Strings.md), PortSwigger Research

---

# External researches on Java products

- FortiNAC - Just a few more RCEs [^FortiNAC-RCEs]

[^FortiNAC-RCEs]: [FortiNAC - Just a few more RCEs](https://frycos.github.io/vulns4free/2023/06/18/fortinac.html), frycos

[^1]: See as an example [The Best Security Is When We All Agree to Keep Everything Secret (Except the Secrets) - NAKIVO Backup & Replication](../../Readwise/Articles/Sonny%20-%20The%20Best%20Security%20Is%20When%20We%20All%20Agree%20to%20Keep%20Everything%20Secret%20(Except%20the%20Secrets)%20-%20NAKIVO%20Backup%20&%20Replication.md)

[^2]: [Sonny - XXE, You Can Depend on Me](../../Readwise/Articles/Sonny%20-%20XXE,%20You%20Can%20Depend%20on%20Me.md)
