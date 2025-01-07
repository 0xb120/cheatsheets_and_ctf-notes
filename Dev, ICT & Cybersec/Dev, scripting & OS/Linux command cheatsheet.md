# Basic Linux Commands

## man

Allows to **read the documentations** of any specific command. `-k` flag allows to **search for a specific keyword**

```bash
kali@kali:~$ man -k passwd
chgpasswd (8) - update group passwords in batch mode
chpasswd (8) - update passwords in batch mode
exim4_passwd (5) - Files in use by the Debian exim4 packages
exim4_passwd_client (5) - Files in use by the Debian exim4 packages
expect_mkpasswd (1) - generate new password, optionally apply it to a user
fgetpwent_r (3) - get passwd file entry reentrantly
getpwent_r (3) - get passwd file entry reentrantly
gpasswd (1) - administer /etc/group and /etc/gshadow
grub-mkpasswd-pbkdf2 (1) - generate hashed password for GRUB
```

## apropos

Is the same as `man -k`

## Create directory structures (mkdir)

```bash
kali@kali:~$ mkdir -p test/{recon,exploit,report}
kali@kali:~$ ls -1 test/
exploit
recon
report
```

# Finding files

## which

Allows to **find a file within the PATH**

```bash
kali@kali:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
kali@kali:~$ which sbd
/usr/bin/sbd
```

## locate

**Search for a specific file reading from a local database** called `locate.db`. To manually update this db it is necessary running `updatedb` command.

```
kali@kali:~$ sudo updatedb
kali@kali:~$ locate sbd.exe
/usr/share/windows-resources/sbd/sbd.exe
```

## find

Used to **find a file recursively though directories**.

```bash
kali@kali:~$ sudo find / -name sbd*
/usr/bin/sbd
/usr/share/doc/sbd
/usr/share/windows-resources/sbd
/usr/share/windows-resources/sbd/sbd.exe
/usr/share/windows-resources/sbd/sbdbg.exe
/var/cache/apt/archives/sbd_1.37-1kali3_amd64.deb
/var/lib/dpkg/info/sbd.md5sums
/var/lib/dpkg/info/sbd.list
```

# Managing services

## Starting, enabling and stopping services

```bash
kali@kali:~$ sudo systemctl start ssh
kali@kali:~$ sudo systemctl enable ssh
```

## List all available services

```bash
kali@kali:~$ systemctl list-unit-files
...
UNIT FILE STATE
proc-sys-fs-binfmt_misc.automount static
-.mount generated
dev-hugepages.mount static
dev-mqueue.mount static
media-cdrom0.mount generated
proc-sys-fs-binfmt_misc.mount static
run-vmblock\\x2dfuse.mount disabled
sys-fs-fuse-connections.mount static
sys-kernel-config.mount static
sys-kernel-debug.mount static
```

## List running services

```bash
kali@kali:~$ sudo ss -antlp | grep sshd
LISTEN 0 128 *:22 *:* users:(("sshd",pid=1343,fd=3))
LISTEN 0 128 :::22 :::* users:(("sshd",pid=1343,fd=4))
```

# Search and show packages

`apt-cache search` command **searches within the “description”** field.

```bash
kali@kali:~$ apt-cache search pure-ftpd
mysqmail-pure-ftpd-logger - real-time logging system in MySQL - Pure-FTPd traffic-logg
pure-ftpd - Secure and efficient FTP server
pure-ftpd-common - Pure-FTPd FTP server (Common Files)
pure-ftpd-ldap - Secure and efficient FTP server with LDAP user authentication
pure-ftpd-mysql - Secure and efficient FTP server with MySQL user authentication
pure-ftpd-postgresql - Secure and efficient FTP server with PostgreSQL user authentica
resource-agents - Cluster Resource Agents

kali@kali:~$ apt show resource-agents
Package: resource-agents
Version: 1:4.2.0-2
...
Description: Cluster Resource Agents
This package contains cluster resource agents (RAs) compliant with the Open
Cluster Framework (OCF) specification, used to interface with various services
in a High Availability environment managed by the Pacemaker resource manager.
.
Agents included:
AoEtarget: Manages ATA-over-Ethernet (AoE) target exports
AudibleAlarm: Emits audible beeps at a configurable interval
...
NodeUtilization: Node Utilization
Pure-FTPd: Manages a Pure-FTPd FTP server instance
Raid1: Manages Linux software RAID (MD) devices on shared storage
...
```

---

# Bash 101

![Bash 101](Bash%20101.md)

# Piping and Redirection

| Stream Name | Description | Numerical Value |
| --- | --- | --- |
| Standard Input (STDIN) | Data fed into the program | 0 |
| Standard Output (STDOUT) | Output from the program (defaults to terminal) | 1 |
| Standard Error (STDERR) | Error messages (defaults to terminal) | 2 |

## Redirecting to a file (> and >>)

```bash
kali@kali:~$ echo "Kali Linux is an open source project" > redirection_test.txt  
kali@kali:~$ cat redirection_test.txt  
Kali Linux is an open source project  
kali@kali:~$ echo "Kali Linux is an open source project again" >> redirection_test.txt  
kali@kali:~$ cat redirection_test.txt  
Kali Linux is an open source project  
Kali Linux is an open source project again
```

## Redirecting from a file (<)

```bash
kali@kali:~$ wc -m < redirection_test.txt  
89
```

## Redirecting STDERR (2>)

```bash
kali@kali:~$ ls .  
Desktop Documents Downloads Music Pictures Public redirection_test.txt Template  
kali@kali:~$ ls ./test  
ls: cannot access '/test': No such file or directory  
kali@kali:~$ ls ./test 2>error.txt  
kali@kali:~$ cat error.txt  
ls: cannot access '/test': No such file or directory
```

## Writing files using cat and redirection

```bash
kali@kali:~$ cat > file.txt << EOF
Here the content of the file
some other contents
still the file
now I can close it
EOF
```

# Text Searching and Manipulation

## grep

**Searches text files for the occurrence of a given regular expression** and outputs any line containing a match to the standard output:

```bash
kali@kali:~$ ls -la /usr/bin | grep zip  
-rwxr-xr-x 3 root root 34480 Jan 29 2017 bunzip2  
-rwxr-xr-x 3 root root 34480 Jan 29 2017 bzip2  
-rwxr-xr-x 1 root root 13864 Jan 29 2017 bzip2recover  
-rwxr-xr-x 2 root root 2301 Mar 14 2016 gunzip  
-rwxr-xr-x 1 root root 105172 Mar 14 2016 gzip

kali@kali:~$ grep -o '[^/]*\.megacorpone\.com' index.html | sort -u > list.txt
```

Removed void lines:

```bash
cat /etc/ssh/sshd_config | grep -v '#' | grep .
```

Other useful flags and examples: 
```bash
# search recursively and case-insensitive
grep -ri "doGet" . 

# search the whole word (-w) and show the matching line-number (-n) only inside .php files
grep -rnw "eval(" . --include *.php --color 

# search a string and only show filenames
grep -rl "test" .
```

Useful regexes:

```bash
# Extract specific JSON fields using regex:
cat file.json | grep -Eo '(json-field-name)":"((\\"|[^"])*)'

# Find misconfigured regexes in PHP:
preg_(match|replace)\s*\(\s*(\[.*?\]|'[^']*'|"[^"]*"|[^\s,]+)  
preg_(match|replace)\(['"][^\/~@;%`#]
```

grep has some other counterparts "on-steroids" that can be used to search also inside other non-textual files:
- [rga](../../Readwise/Articles/httpsgithub.comphiresky%20-%20GitHub%20-%20phireskyripgrep-all%20rga%20ripgrep,%20but%20also%20search%20in%20PDFs,%20E-Books,%20Office%20documents,%20zip,%20tar.gz,%20etc..md) - ripgrep, but also search in PDFs, E-Books, Office documents, zip, tar.gz, etc.
## sed

sed **performs text editing on a stream of text**, either a set of specific files or standard output.

```bash
kali@kali:~$ echo "I need to try hard" | sed 's/hard/harder/'  
I need to try harder

kali@kali:~$ echo $PATH | sed 's/:/\n/g'
/usr/local/bin
/usr/bin
/bin
/usr/games

cat scans/all-tcp-ports.txt | grep open | cut -d'/' -f1 | sed ':a;N;$!ba;s/\n/,/g'
```

## cut

used to **extract a section of text from a line** and output it to the standard output

```bash
kali@kali:~$ echo "I hack binaries,web apps,mobile apps, and just about anything else" | cut -f 2 -d ","  
web apps
```

## awk

**AWK is a programming language designed for text processing** and is typically used as a data extraction and reporting tool.

```bash
kali@kali:~$ echo "hello::there::friend" | awk -F "::" '{print $1, $3}'
hello friend
```

Removed void lines:

```bash
cat /etc/ssh/sshd_config | grep -v '#' | awk 'NF'
```

## sort

## uniq

Make a row unique if the following row is equals to the first one. `-c` count the number of occurrences.

```bash
kali@kali:~$ cat access.log | uniq -c 
```

## wc

Count the number of word in a file

```bash
kali@kali:~$ wc -l access.log
1173 access.log
```

# Comparing Files

## comm

**Compare and shows differences between two files**, dividing them within 3 columns (1-file1 unique rows; 2-file2 unique rows; 3-shared rows)

```bash
kali@kali:~$ cat scan-a.txt  
192.168.1.1  
192.168.1.2  
192.168.1.3  
192.168.1.4  
192.168.1.5  
kali@kali:~$ cat scan-b.txt  
192.168.1.1  
192.168.1.3  
192.168.1.4  
192.168.1.5  
192.168.1.6  
kali@kali:~$ comm scan-a.txt scan-b.txt  
                        192.168.1.1  
192.168.1.2  
                        192.168.1.3  
                        192.168.1.4  
                        192.168.1.5  
            192.168.1.6  
kali@kali:~$ comm -12 scan-a.txt scan-b.txt  
192.168.1.1  
192.168.1.3  
192.168.1.4  
192.168.1.5
```

## diff

Similar to `comm`, but more complex and supports many outputs. `-` and `+` indicates rows which compare only within the first or second arguments.

```bash
kali@kali:~$ diff -c scan-a.txt scan-b.txt
*** scan-a.txt 2018-02-07 14:46:21.557861848 -0700
--- scan-b.txt 2018-02-07 14:46:44.275002421 -0700
***************
*** 1,5 ****
192.168.1.1
- 192.168.1.2
192.168.1.3
192.168.1.4
192.168.1.5
--- 1,5 ----
192.168.1.1
192.168.1.3
192.168.1.4
192.168.1.5
+ 192.168.1.6
kali@kali:~$ diff -u scan-a.txt scan-b.txt
--- scan-a.txt 2018-02-07 14:46:21.557861848 -0700
+++ scan-b.txt 2018-02-07 14:46:44.275002421 -0700
@@ -1,5 +1,5 @@
192.168.1.1
-192.168.1.2
192.168.1.3
192.168.1.4
192.168.1.5
+192.168.1.6
```

## vimdiff

Opens files within various vim tabs and highlights differences between them

# Managing Processes

## Backgrounding processes (&, CTRL+Z and bg)

```bash
kali@kali:~$ ping -c 400 localhost > ping_results.txt &  

kali@kali:~$ ping -c 400 localhost > ping_results.txt  
^Z  
[1]+ Stopped ping -c 400 localhost > ping_results.txt  
kali@kali:~$ bg  
[1]+ ping -c 400 localhost > ping_results.txt  
kali@kali:~$
```

## Jobs Control (jobs and fg)

jobs lists all the josb running in the current terminal session

```bash
kali@kali:~$ ping -c 400 localhost > ping_results.txt
^Z
[1]+ Stopped ping -c 400 localhost > ping_results.txt

kali@kali:~$ find / -name sbd.exe
^Z
[2]+ Stopped find / -name sbd.exe

kali@kali:~$ jobs
[1]- Stopped ping -c 400 localhost > ping_results.txt
[2]+ Stopped find / -name sbd.exe

kali@kali:~$ fg %1
ping -c 400 localhost > ping_results.txt
^C

kali@kali:~$ jobs
[2]+ Stopped find / -name sbd.exe

kali@kali:~$ fg
find / -name sbd.exe
/usr/share/windows-resources/sbd/sbd.exe
```

## Process Control (ps and kill)

`ps` **shows every process** running on the machine

```bash
kali@kali:~$ ps -ef
UID PID PPID C STIME TTY TIME CMD
root 1 0 0 10:18 ? 00:00:02 /sbin/init
root 2 0 0 10:18 ? 00:00:00 [kthreadd]
root 3 2 0 10:18 ? 00:00:00 [rcu_gp]
root 4 2 0 10:18 ? 00:00:00 [rcu_par_gp]
root 5 2 0 10:18 ? 00:00:00 [kworker/0:0-events]
root 6 2 0 10:18 ? 00:00:00 [kworker/0:0H-kblockd]
root 7 2 0 10:18 ? 00:00:00 [kworker/u256:0-events_unbound
root 8 2 0 10:18 ? 00:00:00 [mm_percpu_wq]
root 9 2 0 10:18 ? 00:00:00 [ksoftirqd/0]
root 10 2 0 10:18 ? 00:00:00 [rcu_sched]
```

`kill` **send a signal** (stop, wait, ecc.) to a specific PID

```bash
kali@kali:~$ ps -fC leafpad
UID PID PPID C STIME TTY TIME CMD
kali 1307 938 0 10:57 ? 00:00:00 leafpad

kali@kali:~$ kill 1307
kali@kali:~$ ps aux | grep leafpad
kali 1313 0.0 0.0 6144 888 pts/0 S+ 10:59 0:00 grep leafpad
```

# File and Command Monitoring

## tail

`tail -f` updates the output at runtime

```bash
kali@kali:~$ sudo tail -f /var/log/apache2/access.log
127.0.0.1 - - [02/Feb/2018:12:18:14 -0500] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0
(X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0"
127.0.0.1 - - [02/Feb/2018:12:18:14 -0500] "GET /icons/openlogo-75.png HTTP/1.1" 200
6040 "http://127.0.0.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101
Firefox/52.0"
127.0.0.1 - - [02/Feb/2018:12:18:15 -0500] "GET /favicon.ico HTTP/1.1" 404 500 "-"
"Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0"
```

## watch

`watch` **runs a specific command at regular intervals**

```bash
kali@kali:~$ watch -n 5 w
............
Every 5.0s: w kali: Tue Jan 23 21:06:03 2018
21:06:03 up 7 days, 3:54, 1 user, load average: 0.18, 0.09, 0.03
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
kali tty2 :0 16Jan18 7days 16:29 2.51s /usr/bin/python
```

## head

## less

# Downloading files

## wget

```bash
kali@kali:~$ wget -O report_wget.pdf https://www.offensive-security.com/reports/penetration-testing-sample-report-2013.pdf
--2018-01-28 20:30:04-- https://www.offensive-security.com/reports/penetration-testin
Resolving www.offensive-security.com (www.offensive-security.com)... 192.124.249.5
Connecting to www.offensive-security.com (www.offensive-security.com)|192.124.249.5|:4
HTTP request sent, awaiting response... 200 OK
Length: 27691955 (26M) [application/pdf]
Saving to: ‘report_wget.pdf’

report_wget.pdf 100%[===================>] 26.41M 766KB/s in 28s
2018-01-28 20:30:33 (964 KB/s) - ‘report_wget.pdf’ saved [27691955/27691955]
```

## curl

```bash
kali@kali:~$ curl -o report.pdf <https://www.offensive-security.com/reports/penetration-testing-sample-report-2013.pdf>
% Total % Received % Xferd Average Speed Time Time Time Current
Dload Upload Total Spent Left Speed
100 26.4M 100 26.4M 0 0 1590k 0 0:00:17 0:00:17 --:--:-- 870k
```

## axel

```bash
kali@kali:~$ axel -a -n 20 -o report_axel.pdf https://www.offensive-
security.com/reports/penetration-testing-sample-report-2013.pdf
Initializing download: https://www.offensive-security.com/reports/penetration-testing-
File size: 27691955 bytes
Opening output file report_axel.pdf
Starting download
Connection 0 finished
Connection 1 finished
Connection 2 finished
Connection 3 finished
Connection 4 finished
Connection 5 finished
Connection 6 finished
Connection 7 finished
Connection 8 finished
Connection 9 finished
Connection 10 finished
Connection 11 finished
Connection 13 finished
Connection 14 finished
Connection 15 finished
Connection 16 finished
Connection 18 finished
[100%]
[.....................................................................................
..............................................................................] [
11.1MB/s] [00:00]
Downloaded 26.4 Megabyte in 2 seconds. (11380.17 KB/s)
```

# List of timestamp formats

`kali@kali:~/test$ man strftime`

# Generating openssl certificates and SSH keys

```bash
kali@kali:~$ openssl req -newkey rsa:2048 -nodes -keyout myKey.key -x509 -days 362 -out myCert.crt
kali@kali:~$ ssh-keygen
kali@kali:~$ ssh-keygen -b 2048 -t rsa -f /tmp/sshkey -q -N "" # one-liner
```

# mount folders and drives

```
mount -t nfs [-o vers=2] <ip>:<remote_folder> <local_folder> -o nolock

# Example
mkdir /mnt/new_back
mount -t nfs [-o vers=2] 10.12.0.150:/backup /mnt/new_back -o nolock
```

# iptables

```bash
kali@kali:~$ sudo iptables -I INPUT 1 -s 10.11.1.220 -j ACCEPT		# rule n.1 for incoming traffic from 10.11.1.220
kali@kali:~$ sudo iptables -I OUTPUT 1 -d 10.11.1.220 -j ACCEPT		# rule n.1 for outgoing traffic to 10.11.1.220
kali@kali:~$ sudo iptables -Z										# Reset every counter

kali@kali:~$ sudo iptables -vn -L									# Show every rules in every chain
Chain INPUT (policy ACCEPT 1528 packets, 226K bytes)
 pkts bytes target prot opt in out source destination
 1263 51264 ACCEPT all -- * * 10.11.1.220 0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 1323 packets, 191K bytes)
 pkts bytes target prot opt in out source destination
 1314 78300 ACCEPT all -- * * 0.0.0.0/0 10.11.1.220
```

# debugging commands

## Signals & Core Dumps
![Signals & Core Dumps](Signals%20&%20Core%20Dumps.md)

## file

Determine file type.
>[!tip] 
>**file** tests each argument in an attempt to classify it. There are three sets of tests, performed in this order: filesystem tests, magic tests, and language tests. The *first* test that succeeds causes the file type to be printed [^1] .

```bash
$ file /bin/bash                              
/bin/bash: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0ed1ea49ed834eabf308261eb3d3858a6ab88b04, for GNU/Linux 3.2.0, stripped
```

## strings

Print the strings of printable characters in files.

>[!tip]
>For each file given, GNU **strings** prints the printable character sequences that are at least 4 characters long (or the number given with the options below) and are followed by an unprintable character. By default, it only prints the strings from the initialized and loaded sections of object files; for other types of files, it prints the strings from the whole file [^2].

`strings` is mainly useful for determining the contents of non-text files.

```bash
$ strings jeeves
/lib64/ld-linux-x86-64.so.2
libc.so.6
gets
printf
read
malloc
close
open
...
Hello, good sir!
May I have your name? 
Hello %s, hope you have a good day!
flag.txt
Pleased to make your acquaintance. Here's a small gift: %s
...

# Change endiannes to little-endian
$ strings jeeves -e l
```

## strace

Trace system calls and signals.

>[!tip]
>In the simplest case **strace** runs the specified *command* until it exits. It intercepts and records the system calls which are called by a process and the signals which are received by a process. The name of each system call, its arguments and its return value are printed on standard error [^3].

Each line in the trace contains the system call name, followed by its arguments in parentheses and its return value.

```bash
$ strace ./jeeves 
execve("./jeeves", ["./jeeves"], 0x7ffea050f750 /* 57 vars */) = 0
brk(NULL)                               = 0x561608e28000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=134329, ...}) = 0
mmap(NULL, 134329, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f6ead7d2000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0@n\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1839792, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f6ead7d0000
mmap(NULL, 1852680, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f6ead60b000
mprotect(0x7f6ead630000, 1662976, PROT_NONE) = 0
mmap(0x7f6ead630000, 1355776, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x25000) = 0x7f6ead630000
mmap(0x7f6ead77b000, 303104, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x170000) = 0x7f6ead77b000
mmap(0x7f6ead7c6000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ba000) = 0x7f6ead7c6000
mmap(0x7f6ead7cc000, 13576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f6ead7cc000
close(3)                                = 0
arch_prctl(ARCH_SET_FS, 0x7f6ead7d1540) = 0
mprotect(0x7f6ead7c6000, 12288, PROT_READ) = 0
mprotect(0x56160844a000, 4096, PROT_READ) = 0
mprotect(0x7f6ead81d000, 4096, PROT_READ) = 0
munmap(0x7f6ead7d2000, 134329)          = 0
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(0x88, 0), ...}) = 0
brk(NULL)                               = 0x561608e28000
brk(0x561608e49000)                     = 0x561608e49000
write(1, "Hello, good sir!\n", 17Hello, good sir!
)      = 17
fstat(0, {st_mode=S_IFCHR|0600, st_rdev=makedev(0x88, 0), ...}) = 0
write(1, "May I have your name? ", 22May I have your name? )  = 22
read(0, maoutis
"maoutis\n", 1024)              = 8
write(1, "Hello maoutis, hope you have a g"..., 41Hello maoutis, hope you have a good day!
) = 41
exit_group(0)                           = ?
+++ exited with 0 +++
```

## ltrace

A library call tracer.

>[!hint]
> **ltrace** is a program that simply runs the specified *command* until it exits. It intercepts and records the dynamic library calls which are called by the executed process and the signals which are received by that process [^4].

It can also intercept and print the **system calls** executed by the program (like `strace`).

```bash
$ ltrace ./batcomputer 
setvbuf(0x7f5354d94980, 0, 2, 0)                                                               = 0
setvbuf(0x7f5354d956a0, 0, 2, 0)                                                               = 0
memset(0x7ffc9fe18fd4, '\0', 16)                                                               = 0x7ffc9fe18fd4
printf("Welcome to your BatComputer, Bat"...Welcome to your BatComputer, Batman. What would you like to do?
1. Track Joker
2. Chase Joker
> )                                                  = 96
__isoc99_scanf(0x56063abbc069, 0x7ffc9fe18fd0, 0, 01
)                                           = 1
printf("It was very hard, but Alfred man"..., 0x7ffc9fe18fe4It was very hard, but Alfred managed to locate him: 0x7ffc9fe18fe4
)                                  = 67
memset(0x7ffc9fe18fd4, '\0', 16)                                                               = 0x7ffc9fe18fd4
printf("Welcome to your BatComputer, Bat"...Welcome to your BatComputer, Batman. What would you like to do?
1. Track Joker
2. Chase Joker
> )                                                  = 96
__isoc99_scanf(0x56063abbc069, 0x7ffc9fe18fd0, 0, 03
)                                           = 1
puts("Too bad, now who's gonna save Go"...Too bad, now who's gonna save Gotham? Alfred?
)                                                    = 46
+++ exited (status 0) +++
```

## xxd

Make a hexdump or do the reverse.

>[!tip]
>**xxd** creates a hex dump of a given file or standard input. It can also convert a hex dump back to its original binary form. [...] Moreover, it can be used to perform binary file patching [^5].

```bash
$ cat main.wasm | xxd
00000000: 0061 736d 0100 0000 0105 0160 0001 7f03  .asm.......`....
00000010: 0201 0004 0501 7001 0101 0503 0100 1006  ......p.........
00000020: 1903 7f01 4180 80c0 000b 7f00 4180 80c0  ....A.......A...
00000030: 000b 7f00 4180 80c0 000b 072c 0406 6d65  ....A......,..me
00000040: 6d6f 7279 0200 0469 6e66 6f00 000a 5f5f  mory...info...__
00000050: 6461 7461 5f65 6e64 0301 0b5f 5f68 6561  data_end...__hea
00000060: 705f 6261 7365 0302 0a06 0104 0041 000b  p_base.......A..
00000070: 00d5 d016 0b2e 6465 6275 675f 696e 666f  ......debug_info
00000080: f3a0 0200 0400 0000 0000 0401 0000 0000  ................
00000090: 1c00 3900 0000 0000 0000 5000 0000 0000  ..9.......P.....
000000a0: 0000 400a 0100 0281 0000 0002 8600 0000  ..@.............
000000b0: 028a 0000 0003 0000 0000 0d00 0000 07ed  ................
000000c0: 0300 0000 009f 463e 0300 8c3e 0300 016d  ......F>...>...m
...
```

## hexdump

ascii, decimal, hexadecimal, octal dump.

>[!warning]
>The **hexdump** utility is a filter which displays the specified files, or the standard input, if no files are specified, in a user specified format [^6].

```bash
$ hexdump main.wasm
0000000 6100 6d73 0001 0000 0501 6001 0100 037f
0000010 0102 0400 0105 0170 0101 0305 0001 0610
0000020 0319 017f 8041 c080 0b00 007f 8041 c080
0000030 0b00 007f 8041 c080 0b00 2c07 0604 656d
0000040 6f6d 7972 0002 6904 666e 006f 0a00 5f5f
0000050 6164 6174 655f 646e 0103 5f0b 685f 6165
0000060 5f70 6162 6573 0203 060a 0401 4100 0b00
0000070 d500 16d0 2e0b 6564 7562 5f67 6e69 6f66
0000080 a0f3 0002 0004 0000 0000 0104 0000 0000
0000090 001c 0039 0000 0000 0000 0050 0000 0000
...

$ hexdump main.wasm -C
00000000  00 61 73 6d 01 00 00 00  01 05 01 60 00 01 7f 03  |.asm.......`....|
00000010  02 01 00 04 05 01 70 01  01 01 05 03 01 00 10 06  |......p.........|
00000020  19 03 7f 01 41 80 80 c0  00 0b 7f 00 41 80 80 c0  |....A.......A...|
00000030  00 0b 7f 00 41 80 80 c0  00 0b 07 2c 04 06 6d 65  |....A......,..me|
00000040  6d 6f 72 79 02 00 04 69  6e 66 6f 00 00 0a 5f 5f  |mory...info...__|
00000050  64 61 74 61 5f 65 6e 64  03 01 0b 5f 5f 68 65 61  |data_end...__hea|
00000060  70 5f 62 61 73 65 03 02  0a 06 01 04 00 41 00 0b  |p_base.......A..|
00000070  00 d5 d0 16 0b 2e 64 65  62 75 67 5f 69 6e 66 6f  |......debug_info|
00000080  f3 a0 02 00 04 00 00 00  00 00 04 01 00 00 00 00  |................|
00000090  1c 00 39 00 00 00 00 00  00 00 50 00 00 00 00 00  |..9.......P.....|
...
```

## ldd

`ldd` allows to list the dynamic libraries referenced to a binary

```bash
$ ldd /bin/bash
        linux-vdso.so.1 (0x00007ffc92ade000)
        libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007f135a859000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f135a854000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f135a693000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f135a9ca000)
```



[^1]: https://linux.die.net/man/1/file](https://linux.die.net/man/1/file
[^2]: https://linux.die.net/man/1/strings](https://linux.die.net/man/1/strings
[^3]: https://linux.die.net/man/1/strace](https://linux.die.net/man/1/strace
[^4]: https://linux.die.net/man/1/ltrace](https://linux.die.net/man/1/ltrace
[^5]: https://linux.die.net/man/1/xxd](https://linux.die.net/man/1/xxd
[^6]: https://linux.die.net/man/1/hexdump](https://linux.die.net/man/1/hexdump