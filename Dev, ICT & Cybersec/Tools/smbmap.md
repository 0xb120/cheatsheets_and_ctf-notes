---
Description: SMBMap is a handy SMB enumeration tool
URL: https://github.com/ShawnDEvans/smbmap
---

>[!summary]
>SMBMap allows users to enumerate samba share drives across an entire domain. List share drives, drive permissions, share contents, upload/download functionality, file name auto-download pattern matching, and even execute remote commands. This tool was designed with pen testing in mind, and is intended to simplify searching for potentially sensitive data across large networks.

### Usage

```bash
root@kali:~# smbmap -h
usage: smbmap.py [-h] (-H HOST | --host-file FILE) [-u USERNAME] [-p PASSWORD]
                 [-s SHARE] [-d DOMAIN] [-P PORT] [-x COMMAND] [-L | -R [PATH]
                 | -r [PATH]] [-A PATTERN] [-q] [-F PATTERN]
                 [--search-path PATH] [--download PATH] [--upload SRC DST]
                 [--delete PATH TO FILE] [--skip]

SMBMap - Samba Share Enumerator | Shawn Evans - ShawnDEvans@gmail.com
```

### Main arguments

```bash
Main arguments:
  -H HOST               IP of host
  --host-file FILE      File containing a list of hosts
  -u USERNAME           Username, if omitted null session assumed
  -p PASSWORD           Password or NTLM hash
  -s SHARE              Specify a share (default C$), ex 'C$'
  -d DOMAIN             Domain name (default WORKGROUP)
  -P PORT               SMB port (default 445)
  -v                    Return the OS version of the remote host
  --admin               Just report if the user is an admin
```

### Command execution

```bash
Command Execution:
  Options for executing commands on the specified host

  -x COMMAND            Execute a command ex. 'ipconfig /all'
  --mode CMDMODE        Set the execution method, wmi or psexec, default wmi
```

### Shared drive search

```bash
Shard drive Search:
  Options for searching/enumerating the share of the specified host(s)

  -L                    List all drives on the specified host
  -R [PATH]             Recursively list dirs, and files (no share\path lists ALL shares), ex. 'C$\Finance'
  -r [PATH]             List contents of directory, default is to list root of all shares, ex. -r 'C$\Documents and Settings\Administrator\Documents'
  -A PATTERN            Define a file name pattern (regex) that auto downloads a file on a match (requires -R or -r), not case sensitive, ex '(web|global).(asax|config)'
  -g                    Make the output grep friendly, used with -r or -R (otherwise it outputs nothing)
  --dir-only            List only directories, ommit files.
  --no-write-check      Skip check to see if drive grants WRITE access.
  -q                    Quiet verbose output. Only shows shares you have READ or WRITE on, and suppresses file listing when performing a search (-A).
  --depth DEPTH         Traverse a directory tree to a specific depth. Default is 5.
  --exclude SHARE [SHARE ...]
                        Exclude share(s) from searching and listing, ex. --exclude ADMIN$ C$'
```

### File content search

```bash
File Content Search:
  Options for searching the content of files (must run as root)

  -F PATTERN            File content search, -F '[Pp]assword' (requires admin access to execute commands, and PowerShell on victim host)
  --search-path PATH    Specify drive/path to search (used with -F, default C:\Users), ex 'D:\HR\'
  --search-timeout TIMEOUT
                        Specifcy a timeout (in seconds) before the file search job gets killed. Default is 300 seconds.
```

### File system interaction

```bash
Filesystem interaction:
  Options for interacting with the specified host's filesystem

  --download PATH       Download a file from the remote system, ex.'C$\temp\passwords.txt'
  --upload SRC DST      Upload a file to the remote system ex. '/tmp/payload.exe C$\temp\payload.exe'
  --delete PATH TO FILE
                        Delete a remote file, ex. 'C$\temp\msf.exe'
  --skip                Skip delete file confirmation prompt
```

### Obtain information on the target

```bash
root@kali:~# smbmap -u victim -p s3cr3t -H 192.168.86.61
[+] Finding open SMB ports....
[+] User SMB session establishd on 192.168.86.61...
[+] IP: 192.168.86.61:445   Name: win7-x86.lan
    Disk                                                    Permissions
    ----                                                    -----------
    ADMIN$                                              NO ACCESS
    C$                                                  NO ACCESS
    IPC$                                                NO ACCESS
    Users                                               READ ONLY
	
root@kali:~# smbmap -R Replication -H 10.10.10.100 --no-write-check --depth 10
```

### List shared folders

```bash
smbmap -H <IP> [-P <PORT>] # Null user
smbmap -u "username" -p "password" -H <IP> [-P <PORT>] # Creds
smbmap -u "username" -p "<NT>:<LM>" -H <IP> [-P <PORT>] # Pass-the-Hash
```

### Connect to shared folders

```bash
# List with smbmap, without folder it list everything
smbmap [-u "username" -p "password"] -R [Folder] -H <IP> [-P <PORT>] # Recursive list
smbmap [-u "username" -p "password"] -r [Folder] -H <IP> [-P <PORT>] # Non-Recursive list
smbmap -u "username" -p "<NT>:<LM>" [-r/-R] [Folder] -H <IP> [-P <PORT>] # Pass-the-Hash
```

### Download files

```bash
# Search a file and download
sudo smbmap -R Folder -H <IP> -A . -q # Search the file in recursive mode and download it inside local folder
```