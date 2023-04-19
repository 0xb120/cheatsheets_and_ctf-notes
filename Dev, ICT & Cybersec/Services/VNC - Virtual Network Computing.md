---
Ports: 5800, 5801, 5900, 5901
Description: Is a graphical desktop-sharing system that uses the Remote Frame Buffer protocol (RFB) to remotely control another computer.
---

>[!info]
> In computing, Virtual Network Computing (VNC) is a graphical desktop-sharing system that uses the Remote Frame Buffer protocol (RFB) to remotely control another computer. It transmits the keyboard and mouse events from one computer to another, relaying the graphical-screen updates back in the other direction, over a network.


# Basic Usage

## Connect with known creds

```bash
vncviewer [-passwd passwd.txt] <IP>::5901
```

---

# Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap -sV --script vnc-info,realvnc-auth-bypass,vnc-title -p <PORT> <IP>
```

- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)

```bash
msf> use auxiliary/scanner/vnc/vnc_none_auth
```

---

# Exploitation

## Brute Force

- [medusa](../Tools/medusa.md)
- [hydra](../Tools/hydra.md#VNC%20Brute-Force)

---

# Post Exploitation

## Dump VNC hash

**RealVNC**
`HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\vncserver`
Value: Password

**TightVNC**
`HKEY_CURRENT_USER\Software\TightVNC\Server`
Value: Password or PasswordViewOnly

**TigerVNC**
`HKEY_LOCAL_USER\Software\TigerVNC\WinVNC4`
Value: Password

**UltraVNC**
`C:\Program Files\UltraVNC\ultravnc.ini`
Value: passwd or passwd2

## Decrypting VNC password

Default password is stored in: `~/.vnc/passwd`

```bash
make
vncpwd <vnc password file>
```

- [vncpwd-linux](https://github.com/jeroennijhof/vncpwd)
- [vncpwd-win](https://firebasestorage.googleapis.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-M5Wo4Gb1pdARVZRljw4%2F-M5WoWbSbqt6WK8zsGlT%2Fvncpwd.zip?alt=media&token=13a73ab2-238c-4ba7-89d9-a0280aa3f6fc)