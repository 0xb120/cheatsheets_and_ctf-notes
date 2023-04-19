---
Ports: 69 (UDP)
Description: Is a simple lockstep File Transfer Protocol which allows a client to get a file from or put a file onto a remote host. One of its primary uses is in the early stages of nodes booting from a local area network. TFTP has been used for this application because it is very simple to implement.
---

>[!info]
> **TFTP** uses UDP port 69 and **requires no authentication**—clients read from, and write to servers using the datagram format outlined in RFC 1350. Due to deficiencies within the protocol (namely lack of authentication and no transport security), it is uncommon to find servers on the public Internet. Within large internal networks, however, TFTP is used to serve configuration files and ROM images to VoIP handsets and other devices.

---

# Enumeration

TFTP doesn't provide directory listing so the script `tftp-enum` from [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine)) will try to brute-force default paths.

```
nmap -n -Pn -sU -p69 -sV --script tftp-enum <IP>
```

## Download & Upload

You can use Metasploit or Python to check if you can download/upload files:

```python
import tftpy
client = tftpy.TftpClient(<ip>, <port>)
client.download("filename in server", "/tmp/filename", timeout=5)
client.upload("filename to upload", "/local/path/file", timeout=5)
```

```bash
msf5> auxiliary/admin/tftp/tftp_transfer_util
```

```bash
┌──(kali㉿kali)-[~/…/lab/ntwk/WORKGROUP/10.11.1.111]
└─$ tftp 10.11.1.111
tftp> PUT local_file
tftp> GET conf.txt

TFTP Solarwinds
> tftp – i <IP> GET /etc/passwd (old Solaris)

TFTP Bruteforcing
> TFTP bruteforcer
> Cisco-Torch
```

## Bypass directory with spaces

Use `/X` dir's flag. This displays the short names generated for non-8dot3 file names. The format is that of /N with the short name inserted before the long name. If no short name is present, blanks are displayed in its place.

```powershell
C:\Program Files\Microsoft SQL Server\MSSQL14.SQLEXPRESS\MSSQL>dir /x
 Volume in drive C has no label.
 Volume Serial Number is 8086-3796

 Directory of C:\Program Files\Microsoft SQL Server\MSSQL14.SQLEXPRESS\MSSQL

05/20/2019  01:36 PM    <DIR>                       .
05/20/2019  01:36 PM    <DIR>                       ..
05/20/2019  01:36 PM    <DIR>                       Backup
05/20/2019  01:36 PM    <DIR>                       Binn
05/20/2019  01:36 PM    <DIR>                       DATA
05/20/2019  01:35 PM    <DIR>                       Install
05/20/2019  01:36 PM    <DIR>                       JOBS
01/21/2020  09:09 PM    <DIR>                       Log
08/22/2017  10:16 PM           100,536 SQL_EN~1.DLL sql_engine_core_inst_keyfile.dll
05/20/2019  01:36 PM    <DIR>          TEMPLA~1     Template Data
               1 File(s)        100,536 bytes
               9 Dir(s)   9,691,648,000 bytes free
```

```bash
$ tftp 10.11.1.111
tftp> get ../PROGRA~1/MICROS~1/MSSQL1~1.SQL/MSSQL/TEMPLA~1/master.mdf
```

---

# Misc

- [TFTP dynamic server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#TFTP)
- [TFTP file transfer](../Web%20&%20Network%20Hacking/File%20transfer.md#TFTP)