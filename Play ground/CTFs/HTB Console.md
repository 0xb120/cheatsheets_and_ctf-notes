---
Category:
  - Pwn
Difficulty: Easy
Platform: HackTheBox
Status: 1. In progress
---
>[!quote]
> *Check out the all new HTB Console! Don't try to pwn it though.*


# Set up

No setup required

# Information Gathering

## Binary type and protections:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/htb-console]
└─$ file htb-console
htb-console: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=575e4055094a7f059c67032dd049e4fdbb171266, for GNU/Linux 3.2.0, stripped

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/htb-console]
└─$ checksec --file=htb-console
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols        No    0               3               htb-console
```

## Binary workflow

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/htb-console]
└─$ ./htb-console
Welcome HTB Console Version 0.1 Beta.
>> id
guest(1337) guest(1337) HTB(31337)
>> ls
- Boxes
- Challenges
- Endgames
- Fortress
- Battlegrounds
>> exit
Unrecognized command.
>> quit
Unrecognized command.
>> close
Unrecognized command.
>> ^C
```

## Decompiled code

Binary is stripped, but we can identify the `main` function from the start of the `.text` program trees (the program entry point):

```c
void entry(undefined8 param_1,undefined8 param_2,undefined8 param_3)

{
  undefined8 in_stack_00000000;
  undefined auStack8 [8];
  
  __libc_start_main(FUN_00401397,in_stack_00000000,&stack0x00000008,FUN_00401410,FUN_00401480,
                    param_3,auStack8);
  do {
                    /* WARNING: Do nothing block with infinite loop */
  } while( true );
}
```

- **main.c (0x401397)**
    
    ```c
    void main(void)
    
    {
      char cmd [16];
      
      setup();
      puts("Welcome HTB Console Version 0.1 Beta.");
      do {
        printf(">> ");
        fgets(cmd,0x10,stdin);
        exec-prompt(cmd);
        memset(cmd,0,0x10);
      } while( true );
    }
    ```
    
- **exec-cmd.c (0x401201)**
    
    ```c
    void exec-prompt(char *cmd)
    
    {
      int val_strcmp;
      char flag [16];
      
      val_strcmp = strcmp(cmd,"id\n");
      if (val_strcmp == 0) {
        puts("guest(1337) guest(1337) HTB(31337)");
      }
      else {
        val_strcmp = strcmp(cmd,"dir\n");
        if (val_strcmp == 0) {
          puts("/home/HTB");
        }
        else {
          val_strcmp = strcmp(cmd,"flag\n");
          if (val_strcmp == 0) {
            printf("Enter flag: ");
            fgets(flag,0x30,stdin);
            puts("Whoops, wrong flag!");
          }
          else {
            val_strcmp = strcmp(cmd,"hof\n");
            if (val_strcmp == 0) {
              puts("Register yourself for HTB Hall of Fame!");
              printf("Enter your name: ");
              fgets(&name,10,stdin);
              puts("See you on HoF soon! :)");
            }
            else {
              val_strcmp = strcmp(cmd,"ls\n");
              if (val_strcmp == 0) {
                puts("- Boxes");
                puts("- Challenges");
                puts("- Endgames");
                puts("- Fortress");
                puts("- Battlegrounds");
              }
              else {
                val_strcmp = strcmp(cmd,"date\n");
                if (val_strcmp == 0) {
                  system("date");
                }
                else {
                  puts("Unrecognized command.");
                }
              }
            }
          }
        }
      }
      return;
    }
    ```
    

# The Bug - Stack Buffer Overflow

The “flag” command is vulnerable to Buffer Overflow. The program allocates 16 bytes for the `flag` array but then it copies 48 bytes through the `fgets(flag,0x30,stdin);`.

Because **NX** is enabled it is not possible to execute code from stack, however it is possible to identify the address of `system("date");` and performs a ret2system in order to execute custom code.

# Exploitation

Extracted `system()` address: `0x401381`

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/htb-console]
└─$ gdb -q htb-console
pwndbg: loaded 192 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from htb-console...
(No debugging symbols found in htb-console)
pwndbg> b *0x00401201
Breakpoint 1 at 0x401201
pwndbg> r
...
pwndbg> x/100ig 0x00401201
...
0x401371:    call   0x401090 <strcmp@plt>
0x401376:    test   eax,eax
0x401378:    jne    0x401388
0x40137a:    lea    rdi,[rip+0xd86]        # 0x402107
0x401381:    call   0x401040 <system@plt>
0x401386:    jmp    0x401394
0x401388:    lea    rdi,[rip+0xd7d]        # 0x40210c
0x40138f:    call   0x401030 <puts@plt>
```

Extracted available `pop` gadgets:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/htb-console]
└─$ ropper -f htb-console --search pop
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop

[INFO] File: htb-console
0x000000000040146c: pop r12; pop r13; pop r14; pop r15; ret;
0x000000000040146e: pop r13; pop r14; pop r15; ret;
0x0000000000401470: pop r14; pop r15; ret;
0x0000000000401472: pop r15; ret;
0x000000000040146b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret;
0x000000000040146f: pop rbp; pop r14; pop r15; ret;
0x000000000040117d: pop rbp; ret;
0x0000000000401473: pop rdi; ret;
0x0000000000401471: pop rsi; pop r15; ret;
0x000000000040146d: pop rsp; pop r13; pop r14; pop r15; ret;
```

# Flag