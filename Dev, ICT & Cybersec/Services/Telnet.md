---
Ports: 23
Description: Telnet is a network protocol that gives users a UNsecure way to access a computer over a network.
---

# Enumeration

- [netcat](../Tools/netcat.md)

```bash
nc -vn <IP> 23
```

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap -n -sV -Pn --script "*telnet* and safe" -p 23 <IP>
```

The script `telnet-ntlm-info.nse` will obtain NTLM info (Windows versions).

---

# Misc

## Config files

```
/etc/inetd.conf
/etc/xinetd.d/telnet
/etc/xinetd.d/stelnet
```