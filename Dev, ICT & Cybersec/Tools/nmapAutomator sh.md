---
Description: The main goal for this script is to automate the process of enumeration & recon that is run every time, and instead focus our attention on real pentesting.
URL: https://github.com/21y4d/nmapAutomator
---

```bash
┌──(kali㉿kali)-[/opt/enum/nmapAutomator]
└─$ ./nmapAutomator.sh 

Usage: ./nmapAutomator.sh -H/--host <TARGET-IP> -t/--type <TYPE> [-d/--dns <DNS SERVER>]

Scan Types:
        Quick: Shows all open ports quickly (~15 seconds)
        Basic: Runs Quick Scan, then runs a more thorough scan on found ports (~5 minutes)
        UDP  : Runs "Basic" on UDP ports "requires sudo" (~5 minutes)
        Full : Runs a full range port scan, then runs a thorough scan on new ports (~5-10 minutes)
        Vulns: Runs CVE scan and nmap Vulns scan on all found ports (~5-15 minutes)
        Recon: Suggests recon commands, then prompts to automatically run them
        All  : Runs all the scans (~20-30 minutes)
```

## Summary

The main goal for this script is to automate the process of enumeration & recon that is run every time, and instead focus our attention on real pentesting.

This will ensure two things:

1. Automate nmap scans.
2. Always have some recon running in the background.

Once initial ports are found '*in 5-10 seconds*', we can start manually looking into those ports, and let the rest run in the background with no interaction from our side whatsoever.

## Scans

1. **Network** : Shows all live hosts in the host's network (~15 seconds)
2. **Port** : Shows all open ports (~15 seconds)
3. **Script** : Runs a script scan on found ports (~5 minutes)
4. **Full** : Runs a full range port scan, then runs a thorough scan on new ports (~5-10 minutes)
5. **UDP** : Runs a UDP scan "requires sudo" (~5 minutes)
6. **Vulns** : Runs CVE scan and nmap Vulns scan on all found ports (~5-15 minutes)
7. **Recon** : Suggests recon commands, then prompts to automatically run them
8. **All** : Runs all the scans (~20-30 minutes)

>[!note]
>This is a reconnaissance tool, and it does not perform any exploitation.*

## **Examples**

```bash
./nmapAutomator.sh --host 10.1.1.1 --type All
./nmapAutomator.sh -H 10.1.1.1 -t Basic
./nmapAutomator.sh -H academy.htb -t Recon -d 1.1.1.1
```