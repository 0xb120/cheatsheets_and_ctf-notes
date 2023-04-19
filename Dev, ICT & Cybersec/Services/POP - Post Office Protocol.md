---
Ports: 119, 995
Description: Post Office Protocol (POP) is a type of computer networking and Internet standard protocol that extracts and retrieves email from a remote mail server for access by the host machine. POP is an application layer protocol in the OSI model that provides end users the ability to fetch and receive email.
---

>[!info]
> The POP clients generally connect, retrieve all messages, store them on the client system, and delete them from the server. There are 3 versions of POP, but POP3 is the most used one.


# Basic Usage

## POP syntax

From [here](http://sunnyoasis.com/services/emailviatelnet.html)

```bash
POP commands:
  USER uid           Log in as "uid"
  PASS password      Substitue "password" for your actual password
  STAT               List number of messages, total mailbox size
  LIST               List messages and sizes
  RETR n             Show message n
  DELE n             Mark message n for deletion
  RSET               Undo any changes
  QUIT               Logout (expunges messages if no RSET)
  TOP msg n          Show first n lines of message number msg
  CAPA               Get capabilities
```

---

# Enumeration

## Banner grabbing

- [netcat](../Tools/netcat.md)

```bash
nc -nv <IP> 110
openssl s_client -connect <IP>:995 -crlf -quiet
```

## Automated Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap --scripts "pop3-capabilities or pop3-ntlm-info" -sV -port <PORT> <IP> #All are default scripts
```

- [Bash 101](../Dev,%20scripting%20&%20OS/Bash%20101.md) + [netcat](../Tools/netcat.md)

```
root@kali:~$#$ for user in marcus john mailadmin jenny ryuu joe45; do   ( echo USER ${user}; sleep 2s; echo PASS abcd; sleep 2s; echo LIST; sleep 2s; echo quit) | nc -nvC 10.11.1.72 110;  done
(UNKNOWN) [10.11.1.72] 110 (pop3) open
+OK beta POP3 server (JAMES POP3 Server 2.3.2) ready 
+OK
+OK Welcome marcus
+OK 0 0
.
+OK Apache James POP3 Server signing off.
(UNKNOWN) [10.11.1.72] 110 (pop3) open
+OK beta POP3 server (JAMES POP3 Server 2.3.2) ready 
+OK
+OK Welcome john
...
```

## Manual Enumeration

```bash
root@kali:~# nc -nvC 10.11.1.72 110
(UNKNOWN) [10.11.1.72] 110 (pop3) open
+OK beta POP3 server (JAMES POP3 Server 2.3.2) ready 
ryuu
-ERR
USER ryuu
+OK
PASS abcd
+OK Welcome ryuu
LIST
+OK 2 1807
1 786
2 1021
.
RETR 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <19262980.2.1420734423735.JavaMail.root@pop3>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: ryuu@localhost
Received: from localhost ([127.0.0.1])
          by pop3 (JAMES SMTP Server 2.3.2) with SMTP ID 874
          for <ryuu@localhost>;
          Thu, 8 Jan 2015 11:27:01 -0500 (EST)
Date: Thu, 8 Jan 2015 11:27:01 -0500 (EST)
From: mailadmin@localhost
Dear Ryuu,

Here are your ssh creden ...
```

---

# Exploitation

## Brute Force

- [hydra](../Tools/hydra.md#POP%20Brute-Force)
