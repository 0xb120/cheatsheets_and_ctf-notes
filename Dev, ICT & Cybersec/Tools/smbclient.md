---
Description: ftp-like client to access SMB/CIFS resources on servers
URL: https://www.samba.org/samba/docs/current/man-html/smbclient.1.html
---

```bash
root@kali:~$ smbclient
Usage: smbclient [-?EgqBVNkPeC] [-?|--help] [--usage] [-R|--name-resolve=NAME-RESOLVE-ORDER] [-M|--message=HOST] [-I|--ip-address=IP] [-E|--stderr] [-L|--list=HOST] [-m|--max-protocol=LEVEL]
        [-T|--tar=<c|x>IXFvgbNan] [-D|--directory=DIR] [-c|--command=STRING] [-b|--send-buffer=BYTES] [-t|--timeout=SECONDS] [-p|--port=PORT] [-g|--grepable] [-q|--quiet] [-B|--browse]
        [-d|--debuglevel=DEBUGLEVEL] [-s|--configfile=CONFIGFILE] [-l|--log-basename=LOGFILEBASE] [-V|--version] [--option=name=value] [-O|--socket-options=SOCKETOPTIONS] [-n|--netbiosname=NETBIOSNAME]
        [-W|--workgroup=WORKGROUP] [-i|--scope=SCOPE] [-U|--user=USERNAME] [-N|--no-pass] [-k|--kerberos] [-A|--authentication-file=FILE] [-S|--signing=on|off|required] [-P|--machine-pass]
        [-e|--encrypt] [-C|--use-ccache] [--pw-nt-hash] service <password>
```

### Testing for SMB Null Session

```bash
root@kali:~$ smbclient //MOUNT/share -I $ip -N  

kali@kali:~$ smbclient -L 10.11.0.22 --port=4455 --user=Administrator
Unable to initialize messaging context
Enter WORKGROUP\Administrator's password:
Sharename Type Comment
--------- ---- -------
ADMIN$ Disk Remote Admin
C$ Disk Default share
Data Disk
IPC$ IPC Remote IPC
NETLOGON Disk Logon server share
SYSVOL Disk Logon server share
Reconnecting with SMB1 for workgroup listing.
```

### Authenticate using Kerberos

```bash
smbclient --kerberos //ws01win10.domain.com/C$
```

### List shared folders

```bash
smbclient --no-pass -L //<IP> # Null user
smbclient -U 'username[%passwd]' -L [--pw-nt-hash] //<IP> #If you omit the pwd, it will be prompted. With --pw-nt-hash, the pwd provided is the NT hash
```

### Connect to shared folders

```bash
# Connect using smbclient
smbclient --no-pass //<IP>/<Folder>
smbclient -U 'username[%passwd]' -L [--pw-nt-hash] //<IP> #If you omit the pwd, it will be prompted. With --pw-nt-hash, the pwd provided is the NT hash
smbclient -U '%' -N \\\\<IP>\\<SHARE> # null session to connect to a windows share
smbclient -U '<USER>' \\\\<IP>\\<SHARE> # authenticated session to connect to a windows share (you will be prompted for a password)
# Use --no-pass -c 'recurse;ls'  to list recursively with smbclient
```

### Download files

```bash
# Download all
smbclient //<IP>/<share>
> mask ""
> recurse
> prompt
> mget *
# Download everything to current directory
```