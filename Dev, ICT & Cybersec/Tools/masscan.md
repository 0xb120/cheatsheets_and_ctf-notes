---
Description: It is the fastest port scanner available, can scan entire class A or B subnet in few minutes.
---

```bash
root@kali:~# masscan
usage:
masscan -p80,8000-8100 10.0.0.0/8 --rate=10000
scan some web ports on 10.x.x.x at 10kpps
masscan --nmap
list those options that are compatible with nmap
masscan -p80 10.0.0.0/8 --banners -oB <filename>
save results of scan in binary format to <filename>
masscan --open --banners --readscan <filename> -oX <savefile>
read binary scan results in <filename> and save them as xml in <savefile>
```

```bash

root@kali:~# masscan -p80 10.11.1.0/24 --rate=1000 -e tap0 --router-ip 10.11.0.1

Starting masscan 1.0.3 (<http://bit.ly/14GZzcT>) at 2019-03-04 17:15:40 GMT
-- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 256 hosts [1 port/host]
Discovered open port 80/tcp on 10.11.1.14
Discovered open port 80/tcp on 10.11.1.39
Discovered open port 80/tcp on 10.11.1.219
Discovered open port 80/tcp on 10.11.1.227
Discovered open port 80/tcp on 10.11.1.10
Discovered open port 80/tcp on 10.11.1.50
Discovered open port 80/tcp on 10.11.1.234
```