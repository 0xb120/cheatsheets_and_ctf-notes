>[!tip]
>Java compiler is **javac** from Java JDK

- `*.java` = java source code
- `*.class` = compiled java code
- `*.jar` = compressed data archive used to distribute collections of Java classes

```bash
# Compile a Java source code
javac -source 1.8 -target 1.8 test.java

# Create a JAR archive
jar cmvf META-INF/MANIFEST.MF test.jar test.class

# Execute java program
java -jar test.jar
java test.class
```
See as an example:
- [Custom Java deserialization chain](../Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md#Custom%20Java%20deserialization%20chain)

# J2EE sillabus

- **Enterprise Java Bean** (EJB): relatively heavyweight software component that encapsulates the logic of a specifi c business function within the application. EJBs are intended to take care of various technical challenges that application developers must address, such as transactional integrity.
- **Plain Old Java Object** (POJO): an ordinary Java object normally used to denote objects that are user-defined
- **Java Servlet**: object that resides on an application server and receives HTTP requests from clients and returns HTTP responses
- **Java web container**: platform or engine that provides a runtime environment for Java-based web applications (eg. Tomcat, JBoss, etc.)