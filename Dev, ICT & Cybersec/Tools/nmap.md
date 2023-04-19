---
Description: Nmap ("Network Mapper") is a free and open source (license) utility for network discovery and security auditing.
---

>[!important]
>By default nmap scans the **1000 most popular ports** on a given machine, choosing them according to their percentage from the `/usr/share/nmap/nmap-services` file. The default scanning mode is the [TCP SYN scan](../Web%20&%20Network%20Hacking/Type%20of%20scans%20(TCP%20&%20UDP).md#SYN%20scan) if the application have elevated privileges, otherwise default scan is [TCP connect scan](../Web%20&%20Network%20Hacking/Type%20of%20scans%20(TCP%20&%20UDP).md#TCP%20connect%20scan).

## Flags and options
- [Examples | Nmap Network Scanning](https://nmap.org/book/man-examples.html)
- [Command-line Flags](https://nmap.org/book/port-scanning-options.html)
- [Nmap Cheat Sheet: Commands & Examples](https://highon.coffee/blog/nmap-cheat-sheet/)

---

## NSE (Nmap Scripting Engine)

>[!info]
>NSE scripts are located in `/usr/share/nmap/scripts`

### See information about a script

```bash
kali@kali:~$ nmap --script-help=clamav-exec.nse
Starting Nmap 7.70 ( <https://nmap.org> ) at 2019-05-17 13:41 MDT
clamav-exec
Categories: exploit vuln
<https://nmap.org/nsedoc/scripts/clamav-exec.html>
Exploits ClamAV servers vulnerable to unauthenticated clamav comand execution
...
```

### Full vulnerability scan

```bash
kali@kali:~$ sudo nmap --script vuln 10.11.1.10
[sudo] password for kali:
Starting Nmap 7.70 ( <https://nmap.org> )
Pre-scan script results:
| broadcast-avahi-dos:
| Discovered hosts:
| 224.0.0.251
| After NULL UDP avahi packet DoS (CVE-2011-1002).
|_ Hosts are all up (not vulnerable).
Nmap scan report for 10.11.1.10
Host is up (0.099s latency).
Not shown: 999 filtered ports
PORT STATE SERVICE
80/tcp open http
| http-cookie-flags:
| /CFIDE/administrator/enter.cfm:
| CFID:
| httponly flag not set
| CFTOKEN:
| httponly flag not set
| /CFIDE/administrator/entman/index.cfm:
...
```

---

## General Examples

```bash
nmap -sn 192.168.1.1-200								        # Sweep online hosts
nmap -p 80 192.168.1.1-200 -oG web-sweep.txt			        # Port 80 scan and greppable output file
nmap -sT --top-ports 20 192.168.1.1-200 -oG grepble.txt		    # Connection scan on 20 most famous ports and greppable output file
nmap -sS -sU 10.10.10.10								        # SYN and UDP scan at the same time
nmap -sV -A -O 192.168.1.1								        # Software version scan, OS fingerprint and aggressive detection and enumeration
nmap 192.168.1.1-200 -p 80 --script=banner				        # Port 80 scan and NSE scripting
nmap 192.168.1.34 -sU --min-rate 10000                          # Bulk UDP scan
```