---
Description: Scan for open NETBIOS nameservers on a local or remote TCP/IP network
URL: https://www.kali.org/tools/nbtscan/
---

Enumeration for hosts with NetBIOS over TCP (NBT) open

```bash
root@kali:~# nbtscan -r 192.168.1.1/24  
Doing NBT name scan for addresses from 192.168.1.1/24  
  
IP address       NetBIOS Name     Server    User             MAC address        
------------------------------------------------------------------------------  
192.168.1.1      ALICE            <server>  FASTGATE         00:00:00:00:00:00  
192.168.1.25     FASTGATE         <server>  FASTGATE         00:00:00:00:00:00  
192.168.1.35     BOB                        FASTGATE         00:00:00:00:00:00  
192.168.1.255    FASTGATE         <server>  <unknow>         00:00:00:00:00:00
```