---
Ports: 21
Description: The File Transfer Protocol (FTP) is a standard network protocol used for the transfer of computer files between a client and server on a computer network.
---

>[!info]
> The File Transfer Protocol (FTP) is a standard network protocol used for the transfer of computer files between a client and server on a computer network.

# Basic Usage

## FTP misc commands

`USER usernamePASS passwordHELP` The server indicates which commands are supported
`PORT 127,0,0,1,0,80` This will indicate the FTP server to establish a connection with the IP 127.0.0.1 in port 80 (you need to put the 5th char as "0" and the 6th as the port in decimal or use the 5th and 6th to express the port in hex). A**ctive mode**
`EPRT |2|127.0.0.1|80|` This will indicate the FTP server to establish an Extended TCP connection (indicated by "2") with the IP 127.0.0.1 in port 80. This command supports IPv6.
`PASV` This will open a **passive** connection and will indicate the user were he can connects.
`EPSV` **Extended passive** mode
`LIST` This will send the list of files in current folder
`APPE /path/something.txt` This will indicate the FTP to store the data received from a passive connection or from a PORT/EPRT connection to a file. If the filename exists, it will append the data.
`STOR /path/something.txt` Like APPE but it will overwrite the files
`STOU /path/something.txt` Like APPE, but if exists it won't do anything.
`RETR /path/to/file` A passive or a port connection must be establish. Then, the FTP server will send the indicated file through that connection
`REST 6` This will indicate the server that next time it send something using RETR it should start in the 6th byte.
`TYPE i` Set transfer to binary

---

# Enumeration
## Banner Grabbing

```bash
telnet -vn <IP> 21
openssl s_client -connect crossfit.htb:21 -starttls ftp #Get certificate if any
ftp -p <IP> 21
```

## Anonymous login

`anonymous` : `anonynmousanonymous`
`ftp` : `ftp`

```bash
ftp <IP>
>anonymous
>anonymous
>ls -a # List all files (even hidden) (yes, they could be hidden)
>binary #Set transmission to binary instead of ascii
>ascii #Set transmission to ascii instead of binary
>bye #exit
```

## Unauthenticated enumeration

```bash
HELP
214-The following commands are recognized (* =>'s unimplemented):
214-CWD     XCWD    CDUP    XCUP    SMNT*   QUIT    PORT    PASV    
214-EPRT    EPSV    ALLO*   RNFR    RNTO    DELE    MDTM    RMD     
214-XRMD    MKD     XMKD    PWD     XPWD    SIZE    SYST    HELP    
214-NOOP    FEAT    OPTS    AUTH    CCC*    CONF*   ENC*    MIC*    
214-PBSZ    PROT    TYPE    STRU    MODE    RETR    STOR    STOU    
214-APPE    REST    ABOR    USER    PASS    ACCT*   REIN*   LIST    
214-NLST    STAT    SITE    MLSD    MLST    
214 Direct comments to root@drei.work
FEAT
211-Features:
 PROT
 CCC
 PBSZ
 AUTH TLS
 MFF modify;UNIX.group;UNIX.mode;
 REST STREAM
 MLST modify*;perm*;size*;type*;unique*;UNIX.group*;UNIX.mode*;UNIX.owner*;
 UTF8
 EPRT
 EPSV
 LANG en-US
 MDTM
 SSCN
 TVFS
 MFMT
 SIZE
211 End
```

## Automated enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
sudo nmap <IP> -p21 -sV -sC
sudo nmap <IP> -p 21 --script=ftp*
```

- download all files with `wget`

```bash
wget -m ftp://anonymous:anonymous@10.10.10.98 #Donwload all
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98 #Download all
```

## Browser connection

```bash
ftp://anonymous:anonymous@10.10.10.98
```

---

# Exploitation

## Brute Force

- [medusa](../Tools/medusa.md)
- [hydra](../Tools/hydra.md#FTP%20Brute-Force)

## RCE to Web App connected directly to FTP

Note that **if a web application is sending data controlled by a user directly to a FTP server** you can send double URL encode `%0d%0a` (in double URL encode this is `%250d%250a`) bytes and make the FTP server perform arbitrary actions. One of this possible arbitrary actions is to download content from a users controlled server, perform port scanning or try to talk to other plain-text based services (like http).

### Remote File Upload using curl

```bash
sudo curl -T file_to_be_uploaded.txt ftp://mydomain.com/mydirectory/ --user username:password # Remote file upload from CLI
```

---

# Post Exploitation

## conf files

```
ftpusers
ftp.conf
proftpd.conf

C:/Program Files/FileZilla Server/FileZilla Server.xml
C:/Program Files (x86)/FileZilla Server/FileZilla Server.xml
```

---

# External Resources

- [FTP Bounce Attacks](https://www.thesecuritybuddy.com/vulnerabilities/what-is-ftp-bounce-attack/)
- [FTP Bounce Attacks 2](https://book.hacktricks.xyz/pentesting/pentesting-ftp#ftpbounce-attack)