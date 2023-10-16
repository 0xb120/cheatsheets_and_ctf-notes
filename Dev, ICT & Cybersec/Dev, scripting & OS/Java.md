>[!tip]
>Java compiler is **javac** from Java JDK

# Java 101

- `*.java` = java source code
- `*.class` = compiled java code
- `*.jar` = Java Archive, is a compressed data archive used to distribute collections of Java classes
- `*.war` = Web Application Archive, is used to collect multiple JARs and static content, such as HTML, into a single archive
- `*.ear` = Enterprise Application Archive, contain multiple JARs and WARs to consolidate multiple web applications into a single file
- `.do` extension in URL = used for URL mapping scheme in compiled Java code
- `.jsp` = Java Server Pages

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
- **Java web container**: platform or engine that provides a runtime environment for Java-based web applications (eg. Tomcat, JBoss, etc.)

[^servlets]: https://en.wikipedia.org/wiki/Jakarta_Servlet

# Working with Java

>[!tip] Information gathering and enumeration methodology fomr other researchers
>- [Security Code Audit - For Fun and Fails](https://frycos.github.io/vulns4free/2022/05/24/security-code-audit-fails.html), Frycos

- Easy copy for interesting files into a custom directory:
```bash
find . -iname '*.jar' -exec cp {} ALL_JARS \;
```

- Compact all JAR files into one single "huge" JAR: https://frycos.github.io/vulns4free/2022/05/24/security-code-audit-fails.html
- Debug all the things using **Eclipse IDE for Enterprise Java and Web Developers** or **Eclipse IDE for Java Developers**
	- Search or setup a **JDWP** (Java Debug Wire Protocol) interface our Eclipse instance could talk to


## Servlet mappings

We can use [Process Explorer](../Tools/Sysinternals%20Suite.md#Process%20Explorer) (or other similar tools) to gain additional insight into the Java process we are targeting:
- Working path
- Command used to ran the process
- User running the process
- ...

When analyzing Java application it's also always good searching:
- the `WEB-INF` folder → which is the Java’s default configuration folder path
- `web.xml` → a deployment descriptor file which determine how URLs map to servlets, which URLs require authentication, and other information
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

## Decompile Java

>[!tip] Related resources 
>See also [Android 101](../Mobile%20Hacking/Android%20101.md) and [Android Application Security](../Mobile%20Hacking/Android%20Application%20Security.md)

- [JD-GUI](../Tools/JD-GUI.md)
- [jadx](../Tools/jadx.md)
- [Bytecode Viewer](../Tools/Bytecode%20Viewer.md)

---

# External researches on Java products

- [FortiNAC - Just a few more RCEs](https://frycos.github.io/vulns4free/2023/06/18/fortinac.html), frycos