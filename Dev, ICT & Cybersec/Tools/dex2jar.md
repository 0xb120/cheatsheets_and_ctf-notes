---
Description: Dex to Java decompiler
URL: https://github.com/pxb1988/dex2jar
---

>[!summary]
>Tools to work with android .dex and java .class files
>1. **dex-reader/writer**: Read/write the Dalvik Executable (.dex) file. It has a [light weight API similar with ASM](https://sourceforge.net/p/dex2jar/wiki/Faq#markdown-header-want-to-read-dex-file-using-dex2jar).
>2. **d2j-dex2jar**: Convert .dex file to .class files (zipped as jar)
>3. **smali/baksmali**: disassemble dex to smali files and assemble dex from smali files. different implementation to [smali/baksmali](http://code.google.com/p/smali), same syntax, but we support escape in type desc "Lcom/dex2jar\t\u1234;"
>4. other tools: [d2j-decrypt-string](https://sourceforge.net/p/dex2jar/wiki/DecryptStrings)

## Decompile .dex into .jar file

```bash
$ d2j-dex2jar -f -o classes.jar diva-beta.apk
```

>[!warning]
>When converting DEX to JAR, you may get Out of Memory Error for large size DEX file. Here, we need to increase the size of the JVM memory in d2j_invoke script. Change the values accordingly to your system requirements such as -Xmx2048m.
