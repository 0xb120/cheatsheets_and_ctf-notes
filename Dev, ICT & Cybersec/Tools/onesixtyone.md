---
Description: Fast SNMP Scanner. Attempts a SNMP brute-force against a list IP addresses
URL: https://github.com/trailofbits/onesixtyone
---

Attempts a SNMP brute-force against a list IP addresses

```bash
┌──(kali㉿kali)-[/usr/share/seclists/Usernames]
└─$ onesixtyone                                                                          
onesixtyone 0.3.3 [options] <host> <community>
  -c <communityfile> file with community names to try
  -i <inputfile>     file with target hosts
  -o <outputfile>    output log
  -p                 specify an alternate destination SNMP port
  -d                 debug mode, use twice for more information

  -s                 short mode, only print IP addresses

  -w n               wait n milliseconds (1/1000 of a second) between sending packets (default 10)
  -q                 quiet mode, do not print log to stdout, use with -l
host is either an IPv4 address or an IPv4 address and a netmask
default community names are: public private

Max number of hosts :           65535
Max community length:           32
Max number of communities:      16384

examples: onesixtyone 192.168.4.0/24 public
          onesixtyone -c dict.txt -i hosts -o my.log -w 100
```

```bash
kali@kali:~$ onesixtyone -c community.list -i ips.list
Scanning 254 hosts, 3 communities
10.11.1.14 [public] Hardware: x86 Family 6 Model 12 Stepping 2 AT/AT COMPATIBLE -
Software: Windows 2000 Version 5.1 (Build 2600 Uniprocessor Free)
10.11.1.13 [public] Hardware: x86 Family 6 Model 12 Stepping 2 AT/AT COMPATIBLE -
Software: Windows 2000 Version 5.1 (Build 2600 Uniprocessor Free)
10.11.1.22 [public] Linux barry 2.4.18-3 #1 Thu Apr 18 07:37:53 EDT 2002 i686
```