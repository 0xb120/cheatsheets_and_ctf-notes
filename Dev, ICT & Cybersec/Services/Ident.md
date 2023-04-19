---
Ports: 113
Description: s an Internet protocol that helps identify the user of a particular TCP connection.
---

# Enumeration

## Manual - Get user/Identify the service

If a machine is running the service ident and [SMB & NetBIOS](SMB%20&%20NetBIOS.md) samba (445) and you are connected to samba using the port 43218. You can get which user is running the samba service by doing:

```bash
┌──(kali㉿kali)-[~/…/ntwk/thinc.local/10.11.1.136/scans]
└─$ nc -nv 10.11.1.136 113 
(UNKNOWN) [10.11.1.136] 113 (auth) open
445,43218
445 , 43218 : ERROR : NO-USER
22,0
22 , 0 : ERROR : INVALID-PORT

0 , 0 : ERROR : UNKNOWN-ERROR
```

## Automated Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.3p2 Debian 9 (protocol 2.0)
|_auth-owners: root
| ssh-hostkey: 
|   1024 88:23:98:0d:9d:8a:20:59:35:b8:14:12:14:d5:d0:44 (DSA)
|_  2048 6b:5d:04:71:76:78:56:96:56:92:a8:02:30:73:ee:fa (RSA)
113/tcp open  ident
|_auth-owners: identd
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: LOCAL)
|_auth-owners: root
445/tcp open  netbios-ssn Samba smbd 3.0.24 (workgroup: LOCAL)
|_auth-owners: root
```

- Ident-user-enum

```bash
┌──(root💀kali)-[/home/…/lab/ntwk/thinc.local/10.11.1.136]
└─# /usr/bin/ident-user-enum 10.11.1.136 113 22 139 445
ident-user-enum v1.0 ( http://pentestmonkey.net/tools/ident-user-enum )

10.11.1.136:113 identd
10.11.1.136:22  root
10.11.1.136:139 root
10.11.1.136:445 root
```

---

# Files

```
identd.conf
```