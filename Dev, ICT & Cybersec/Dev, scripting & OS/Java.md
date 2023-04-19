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
```