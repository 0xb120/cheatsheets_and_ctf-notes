---
Description: Allows you to enumerate the SNMP devices and places the output in a very human readable friendly format. It could be useful for penetration testing or systems monitoring.
URL: https://www.kali.org/tools/snmpcheck/
---

```bash
┌──(kali㉿kali)-[~/Documents/lab]
└─$ snmp-check -h
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

 Usage: snmp-check [OPTIONS] <target IP address>

  -p --port        : SNMP port. Default port is 161;
  -c --community   : SNMP community. Default is public;
  -v --version     : SNMP version (1,2c). Default is 1;

  -w --write       : detect write access (separate action by enumeration);

  -d --disable_tcp : disable TCP connections enumeration!
  -t --timeout     : timeout in seconds. Default is 5;
  -r --retries     : request retries. Default is 1; 
  -i --info        : show script version;
  -h --help        : show help menu;
```

```bash
┌──(kali㉿kali)-[~/Documents/lab]
└─$ snmp-check 10.11.1.177
```