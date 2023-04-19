>[!summary]
>This is the process whereby, once a machine has been compromised and access has been gained as a normal user, an attempt is made to scale up one's user rank to a higher one, until administrative users are reached.

## Understanding systems permissions

Depending on the system being tested, it is important to know the basic functioning of the security and permissions mechanics it implements:

- [Linux Security 101](../Dev,%20scripting%20&%20OS/Linux%20Security%20101.md)
- [Windows Security 101](../Dev,%20scripting%20&%20OS/Windows%20Security%20101.md)

Furthermore is fundamental know **how to perform information gathering** once a shell is obtained, in order to find any possible way to escalate privileges:

- [Linux internal information gathering](Internal%20information%20gathering.md#Linux)
- [Windows internal information gathering](Internal%20information%20gathering.md#Windows)
- [Automated Enumeration Tools](Internal%20information%20gathering.md#Automated%20Enumeration%20Tools)

---

## Privilege Escalation Methodologies

Search for misconfigured services, insufficient file permission restrictions on binaries or services, direct kernel vulnerabilities, vulnerable software running with high privileges, sensitive information stored on local files, registry settings that always elevate privileges before executing a binary, installation scripts that may contain hard coded credentials, and many others.
```start-multi-column
ID: ID_6lik
Number of Columns: 2
Largest Column: standard
```

### Linux Methodologies

- [Linux PrivEsc Methodology Mind Map](Linux%20PrivEsc%20Methodology%20Mind%20Map.md)
- [g0tmi1k basic linux privesc](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
- [HackTricks Linux PrivEsc](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)
- [PayloadsAllTheThings Linux PrivEsc](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md#checklists)
- [Ignitetechnologies PrivEsc examples](https://github.com/Ignitetechnologies/Privilege-Escalation)
- [Infosecsanyam checklist](https://oscp.infosecsanyam.in/priv-escalation/linux-priv-escalation/checklist-linux-privilege-escalation)

--- column-end ---

### Windows Methodologies

- [Windows PrivEsc Methodology Mind Map](Windows%20PrivEsc%20Methodology%20Mind%20Map.md)
- [HackTricks Windows PrivEsc](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)
- [Windows PrivEsc checklist](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md)
- [Infosecsanyam checklist](https://oscp.infosecsanyam.in/priv-escalation/windows-priv-escalation/checklist-local-windows-privilege-escalation)
- [Fuzzysec - Windows Privilege Escalation Fundamentals](https://www.fuzzysecurity.com/tutorials/16.html)
- [absolomb's PrivEsc chetasheet](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)

=== end-multi-column
## External Tools and References

Here are many of the various online cheat sheets and tools dedicated to privilege escalation
```start-multi-column
ID: ID_yfx0
Number of Columns: 2
Largest Column: standard
```

### Linux

- [Automated Enumeration Tools](Internal%20information%20gathering.md#Automated%20Enumeration%20Tools)
- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
- [HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation)
- [GTFOBins](https://gtfobins.github.io/)
- [g0tmi1k basic linux privesc](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
- [0xsp Linux PrivEsc cheatsheet](https://0xsp.com/offensive/privilege-escalation-cheatsheet#toc-12)
- [infosecsanyam](https://oscp.infosecsanyam.in/priv-escalation/linux-priv-escalation)


--- column-end ---

### Windows

- [Automated Enumeration Tools](Internal%20information%20gathering.md#Automated%20Enumeration%20Tools)
- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [HackTricks](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation)
- [LOLBAS](https://lolbas-project.github.io/#)
- [absolomb's PrivEsc chetasheet](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)
- [Fuzzysec - Windows Privilege Escalation Fundamentals](https://www.fuzzysecurity.com/tutorials/16.html)
- [0xsp Windows PrivEsc cheatsheet](https://0xsp.com/offensive/privilege-escalation-cheatsheet#toc-0)
- [emilyanncr post exploitation](https://github.com/emilyanncr/Windows-Post-Exploitation)
- [Empire](../Tools/Empire.md)
- [nishang](https://github.com/samratashok/nishang)
- [powersploit](../Tools/powersploit.md)
- [Sysinternals Suite](../Tools/Sysinternals%20Suite.md)
- [infosecsanyam](https://oscp.infosecsanyam.in/priv-escalation/windows-priv-escalation)

=== end-multi-column
