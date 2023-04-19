---
Description: Metasploit tools under /usr/share/metasploit-framework/tools/
---

## pattern_create.rb

Allows to create a unique pattern (`msf-pattern_create`):

```bash
kali@kali:~$ msf-pattern_create -h
Usage: msf-pattern_create [options]
Example: msf-pattern_create -l 50 -s ABC,def,123
Ad1Ad2Ad3Ae1Ae2Ae3Af1Af2Af3Bd1Bd2Bd3Be1Be2Be3Bf1Bf

Options:
	-l, --length <length> The length of the pattern
	-s, --sets <ABC,def,123> Custom Pattern Sets
	-h, --help Show this message
```

## pattern_offset.rb

Allows to calculate the offset from the pattern (`msf-pattern_offset`):

```bash
kali@kali:~$ msf-pattern_offset -l 800 -q 42306142
[*] Exact match at offset 780
```

## NASM Shell

Allows to convert assembly instruction into op-code:

```bash
kali@kali:~$ msf-nasm_shell
nasm > jmp esp
00000000 FFE4 jmp esp
nasm >
```