---
author: Federico Dotta
aliases:
  - Java Applet + Serialization in 2024! What Could Go Wrong?
  - Pentesting a Java Applet
tags:
  - readwise/articles
url: https://security.humanativaspa.it/java-applet-serialization-in-2024-what-could-go-wrong/
created: 2024-08-21
---
# Java Applet + Serialization in 2024! What Could Go Wrong?

![rw-book-cover](https://security.humanativaspa.it/favicon.ico)

Running and intercepting traffic from legacy [Java Applet](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Java.md#Java%20Applet) in a modern environment is challenging due to browser deprecation. Below is a methodology to run applets using **OpenJDK 8** and intercept their traffic via **Burp Suite**.

## 1. The Runtime Environment

Instead of relying on unstable legacy browsers inside a Virtual Machine, use the standalone `appletviewer` binary included with OpenJDK 8. This version minimizes compatibility issues with older Java compilers.

### **Bypassing the Security Manager**

To ensure the applet runs without restriction, create a permissive security policy file (e.g., `java.policy`) to disable the default Security Manager constraints:

```java
grant {
    permission java.security.AllPermission;
};
```

### **Running the Applet**

Execute the applet by pointing `appletviewer` to the target page URL. Pass the security policy and manager flags as JVM arguments:

```bash
$ /usr/lib/jvm/java-8-openjdk/bin/appletviewer \
  -J-Djava.security.manager \
  -J-Djava.security.policy=java.policy \
  https://<TARGET>/<PAGE>.htm
```

> **Debug Tip:** If the connection fails or hangs, add the SSL debug flag to view the handshake process in detail: `-J-Djavax.net.debug=all`

---

## 2. Traffic Interception (Invisible Proxy Setup)

Since applets often ignore system proxy settings, you must configure [Burpsuite](../../Dev,%20ICT%20&%20Cybersec/Tools/Burpsuite.md) as an **Invisible Proxy**.

### **Step A: DNS Spoofing**

Force the operating system to route the target domain's traffic to your localhost. Edit your hosts file (`/etc/hosts` on Linux or `C:\Windows\System32\drivers\etc\hosts` on Windows):

```txt
127.0.0.1 www.target.com
```

### **Step B: Burp Proxy Configuration**

1. **Listener:** Set up a Burp Proxy listener on port `443` (requires root/admin privileges).
    
2. **Mode:** Enable **Support invisible proxying**.
    
3. **SNI Routing (Crucial):**
    
    - _If the target uses Server Name Indication (SNI):_ Configure the "Redirect to host" field with the **hostname**, not the IP.
    - _If SNI is disabled:_ You can simply redirect to the target IP address.
        

### **Step C: Breaking the DNS Loop**

If you redirect to the _hostname_ (for SNI) while having `127.0.0.1` in your `/etc/hosts` file, Burp will resolve the domain back to itself, creating an infinite loop.

- **Fix:** In Burp Suite (**Network Settings > Hostname Resolution**), manually map `www.target.com` to its **real public IP address**.
    

---

## 3. Establishing Trust (SSL/TLS)

The applet will reject Burp's self-signed certificate. You must import Burp's CA into the specific Java Runtime environment used by `appletviewer`.

1. **Export:** Save the Burp CA certificate in **DER** format, then convert it to **PEM** using OpenSSL.
    
2. **Import:** Use `keytool` to add it to the JVM's trusted store:
    

```bash
$ /usr/lib/jvm/java-8-openjdk/bin/keytool \
  -import -alias burp \
  -keystore /usr/lib/jvm/java-8-openjdk/jre/lib/security/cacerts \
  -file <PATH>/certCABurp.pem
```

> **Note:** Ensure you also enable **all legacy TLS protocols and ciphers** in Burp's TLS settings, as the applet likely relies on outdated encryption standards.

---

## 4. Critical Configuration: HTTP Compression

**⚠️ Important:** Unlike modern browsers, legacy applets (and thick clients) often crash if the HTTP response format differs even slightly from what they expect.

- **The Issue:** Burp Suite automatically decompresses (unzips) HTTP responses by default to make them readable for the tester.
    
- **The Fix:** Go to **Proxy > Miscellaneous** and **disable** automatic decompression. The applet likely requires the raw, compressed stream to function correctly.

## Highlights


> we needed to run the applet. A possible solution would be to run it in an old browser in a VM, but we would leave this option as a last resort. Instead, we installed a version of **OpenJDK 8** and used the **appletviewer** binary to run the applet. We preferred to use a not-very-recent version of OpenJDK to minimize compatibility issues with the old Java version probably used to compile the applet
> [View Highlight](https://read.readwise.io/read/01j5tpgnh4trk8zeat063m11cq)
> #tools 


> e created the following Java policy to **disable the security manager** in the applet (saved in file “java.policy”):
>  grant {
>  permission java.security.AllPermission;
>  };
>  Then, we **ran the applet** as follows (the URL is the URL of the page that starts the applet):
>  $ /usr/lib/jvm/java-8-openjdk/bin/appletviewer -J-Djava.security.manager -J-Djava.security.policy=java.policy https://<TARGET>/<PAGE>.htm
> [View Highlight](https://read.readwise.io/read/01j5tphw71y0h0eare178fehe5)



> One command-line option that helped a lot is “-J-Djavax.net.debug=all”, which prints precious **debug information**:
>  $ /usr/lib/jvm/java-8-openjdk/bin/appletviewer -J-Djava.security.manager -J-Djava.security.policy=java.policy -J-Djavax.net.debug=all https://<TARGET>/<PAGE>.htm
> [View Highlight](https://read.readwise.io/read/01j5tpjkahneg0s3ek3tvtrxxw)



> We set up Burp Proxy as an **invisible proxy**, transparently redirecting the applet traffic to the HTTP port Burp Proxy was listening on.
>  To do this operation we had to first edit the */etc/hosts* file (*C:\Windows\System32\drivers\etc\hosts* in Windows), adding a resolution of our target to 127.0.0.1 (assume *www.target.com* as our target):
>  127.0.0.1 www.target.com
>  Then we could setup an invisible proxy in the Proxy settings of Burp Suite as follows (you may have to run Burp Suite as root to bind on port 443):
>  ![](https://security.humanativaspa.it/sec/wp-content/uploads/2024/02/1.png)
>  ![](https://security.humanativaspa.it/sec/wp-content/uploads/2024/02/2.png)
> [View Highlight](https://read.readwise.io/read/01j5tppkh40d5hj4yqqjpzb1t8)



> **Pay attention**: In the second screenshot, I used the hostname of my target instead of its IP address in the “Redirect to host” field because my target had [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) enabled. To use the hostname, however, we need to add the correct DNS resolution of the hostname inside Burp Suite. Otherwise, it will resolve again to 127.0.0.1 due to our rule in the */etc/hosts* file. If your target doesn’t have SNI enabled, you can simply put the IP address instead of the hostname in the proxy listener configuration. If your target has SNI enabled, you can add a custom DNS resolution in Burp Suite in the network settings:
>  ![](https://security.humanativaspa.it/sec/wp-content/uploads/2024/02/3.png)
>  The HTTP traffic of our applet reached Burp Proxy, but we got a certificate exception in the applet caused by the fact that Burp Suite’s certificate was self-signed and consequently not valid for the applet. To overcome this issue we simply **installed the Burp Suite CA** in the Java Keystore used for CA certificates (Burp Suite CA’s certificate can be exported from the Proxy Listener configuration tab in DER format and then can be converted to PEM format using OpenSSL):
>  $ /usr/lib/jvm/java-8-openjdk/bin/keytool -import -alias example -keystore /usr/lib/jvm/java-8-openjdk/jre/lib/security/cacerts -file <PATH>/certCABurp.pem
>  In order to prevent any further TLS issues we enabled all TLS protocols and ciphers in Burp Suite, including old and unsafe ones:
>  ![](https://security.humanativaspa.it/sec/wp-content/uploads/2024/02/4.png)
> [View Highlight](https://read.readwise.io/read/01j5tpsggf7yyadejf5a3asbej)



> **Burp Suite tip**: when you analyze an applet or more generally an application that is not a web application executed in the browser (such as a client-server application or a mobile application) remember that Burp Suite by default decompresses HTTP responses it receives from the backend to simplify the pentester’s analysis. While browsers do not complain if the content they receive from Burp Suite is not compressed, applets and other applications may break (and usually do). You can disable automatic decompression and other operations executed on HTTP requests and responses by default in the Proxy -> Miscellaneous configuration tab.
> [View Highlight](https://read.readwise.io/read/01j5tpvn6r8nd4te8wdfwkdr67)
> #tools

