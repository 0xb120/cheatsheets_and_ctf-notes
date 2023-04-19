---
Ports: 25, 465, 587
Description: SMTP (Simple Mail Transfer Protocol) is a TCP/IP protocol used in sending and receiving e-mail.
---

>[!info]
> **SMTP (Simple Mail Transfer Protocol)** is a TCP/IP protocol used in **sending** and receiving **e-mail**. However, since it is limited in its ability to queue messages at the receiving end, it is usually used with one of two other protocols, POP3 or IMAP, that let the user save messages in a server mailbox and download them periodically from the server.

In other words, **users typically use** a program that uses **SMTP for sending e-mail** and either **POP3 or IMAP for receiving** e-mail. On Unix-based systems, **sendmail** is the most widely-used SMTP server for e-mail. A commercial package, Sendmail, includes a POP3 server. **Microsoft Exchange** includes an SMTP server and can also be set up to include POP3 support.

# Basic Usage

## SMTP commands
- [SMTP - Commands](https://book.hacktricks.xyz/pentesting/pentesting-smtp/smtp-commands)
- [SMTP commands](https://serversmtp.com/smtp-commands/) from turboSMTP

## Email Headers

If you have the opportunity to **make the victim send you a email** (via contact form of the web page for example), do it because **you could learn about the internal topology** of the victim seeing the headers of the mail.

You can also get an email from a SMTP server trying to **send to that server an email to a non-existent address** (because the server will send to the attacker a NDN mail). But, be sure that you send the email from an allowed address (check the SPF policy) and that you can receive NDN messages.

You should also try to **send different contents because you can find more interesting information** on the headers like: `X-Virus-Scanned: by av.domain.com` You should send the EICAR test file. Detecting the **AV** may allow you to exploit **known vulnerabilities.**


---

# Enumeration

## Banner Grabbing

- [netcat](../Tools/netcat.md)

```bash
# SMTP
nc -vn <IP> 25

# SMTPS
openssl s_client -crlf -connect smtp.mailgun.org:465 #SSL/TLS without starttls command
openssl s_client -starttls smtp -crlf -connect smtp.mailgun.org:587
```

## Finding MX servers of an organisation

```bash
dig +short mx google.com
```

## NTLM Auth - Information disclosure

If the server supports NTLM auth (Windows) you can obtain sensitive info (versions). More info **[here](https://medium.com/@m8r0wn/internal-information-disclosure-using-hidden-ntlm-authentication-18de17675666)**.

```bash
root@kali: telnet example.com 587 
220 example.com SMTP Server Banner 
>> HELO 
250 example.com Hello [x.x.x.x] 
>> AUTH NTLM 334 
NTLM supported 
>> TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA= 
334 TlRMTVNTUAACAAAACgAKADgAAAAFgooCBqqVKFrKPCMAAAAAAAAAAEgASABCAAAABgOAJQAAAA9JAEkAUwAwADEAAgAKAEkASQBTADAAMQABAAoASQBJAFMAMAAxAAQACgBJAEkAUwAwADEAAwAKAEkASQBTADAAMQAHAAgAHwMI0VPy1QEAAAAA
```

## Manual username enumeration

- `RCPT TO`

```bash
$ telnet 10.0.10.1 25
Trying 10.0.10.1...
Connected to 10.0.10.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO x
250 myhost Hello [10.0.0.99], pleased to meet you
MAIL FROM:test@test.org
250 2.1.0 test@test.org... Sender ok
RCPT TO:test
550 5.1.1 test... User unknown
RCPT TO:admin
550 5.1.1 admin... User unknown
RCPT TO:ed
250 2.1.5 ed... Recipient ok
```

- `VRFY`

```bash
$ telnet 10.0.0.1 25
Trying 10.0.0.1...
Connected to 10.0.0.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO
501 HELO requires domain address
HELO x
250 myhost Hello [10.0.0.99], pleased to meet you
VRFY root
250 Super-User <root@myhost>
VRFY blah
550 blah... User unknown
```

- `EXPN`

```bash
$ telnet 10.0.10.1 25
Trying 10.0.10.1...
Connected to 10.0.10.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO
501 HELO requires domain address
HELO x
EXPN test
550 5.1.1 test... User unknown
EXPN root
250 2.1.5 <ed.williams@myhost>
EXPN sshd
250 2.1.5 sshd privsep <sshd@mail2>
```

## Automatic Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))
- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)
- [netcat (SMTP Enumeration)](../Tools/netcat.md#SMTP%20Enumeration)

```bash
# General SMTP Enumeration
nmap -p25 --script smtp-commands 10.10.10.10

# User enumeration
auxiliary/scanner/smtp/smtp_enum
smtp-user-enum -M <MODE> -u <USER> -t <IP>
nmap --script smtp-enum-users <IP>
```

- User Enumeration using python:

```python
#!/usr/bin/python
import socket
import sys

if len(sys.argv) != 2:
	print "Usage vrfy.py <username>"
	sys.exit(0)
# Create socket
s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# Connect to SMNP server
connect = s.connect(('192.168.1.34',25))
# Receive and print the banner
banner=s.recv(1024)
print banner
# VRFY user
s.send('VRFY ' + sys.argv[1] + '\r\n')
result=s.recv(1024)
print result
s.close()
```

## Information extraction from emails

**Delivery Status Notification Reports**: If you send an **email** to an organization to an **invalid address**, the organization will notify that the address was invalided sending a **mail back to you**. **Headers** of the returned email will **contain** possible **sensitive information** (like IP address of the mail services that interacted with the reports or anti-virus software info).

---

# Exploitation

## Brute Force

- [hydra](../Tools/hydra.md#SMTP%20Brute-Force)

---

# Misc

## Send email from linux console

```bash
root@kali:~$ sendEmail -t itdept@victim.com -f techsupport@bestcomputers.com -s 192.168.8.131 -u Important Upgrade Instructions -a /tmp/BestComputers-UpgradeInstructions.pdf
Reading message body from STDIN because the '-m' option was not used.
If you are manually typing in a message:
  - First line must be received within 60 seconds.
  - End manual input with a CTRL-D on its own line.

IT Dept,

We are sending this important file to all our customers. It contains very important instructions for upgrading and securing your software. Please read and let us know if you have any problems.

Sincerely,
```

## Send email from netcat

```bash
┌──(kali㉿kali)-[~/…/ntwk/thinc.local/10.11.1.217/exploit]
└─$ nc -nvC 10.11.1.217 25  
(UNKNOWN) [10.11.1.217] 25 (smtp) open
220 hotline.localdomain ESMTP Postfix
HELO server
250 hotline.localdomain
MAIL FROM:hack@hack.com
250 2.1.0 Ok
RCPT TO:mat@domdom.com
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
This is the body homie
.
250 2.0.0 Ok: queued as 83DB5D2B4A
QUIT
221 2.0.0 Bye
```

## SMTP Authentication

```bash
┌──(kali㉿kali)-[~/…/thinc.local/10.11.1.229/scans/sqli]
└─$ nc -nvC 10.11.1.229 25
(UNKNOWN) [10.11.1.229] 25 (smtp) open
220 MAIL ESMTP
helo server
250 Hello.
auth login
334 VXNlcm5hbWU6
ZXJpY0B0aGluYy5sb2NhbA==
334 UGFzc3dvcmQ6
c3VwM3JzM2NyM3Q=
235 authenticated.
```

## Config files

```
sendmail.cf
submit.cf
```

## Mail spoofing and defense

- [Mail spoofing](https://book.hacktricks.xyz/pentesting/pentesting-smtp#mail-spoofing)
- [SPF](https://book.hacktricks.xyz/pentesting/pentesting-smtp#spf)
- [DKIM](https://book.hacktricks.xyz/pentesting/pentesting-smtp#dkim)
- [DMARC](https://book.hacktricks.xyz/pentesting/pentesting-smtp#dmarc)
- [Tools](https://book.hacktricks.xyz/pentesting/pentesting-smtp#tools)