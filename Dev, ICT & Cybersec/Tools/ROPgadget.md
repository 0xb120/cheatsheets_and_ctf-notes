---
Description: This tool lets you search your gadgets on your binaries to facilitate your ROP exploitation. ROPgadget supports ELF, PE and Mach-O format on x86, x64, ARM, ARM64, PowerPC, SPARC and MIPS architectures.
URL: https://github.com/JonathanSalwan/ROPgadget
---

```bash
$ ROPgadget --binary software.bin
...
0x000000000043e21f : adc al, 0 ; add byte ptr [rax - 0x7d], cl ; ret 0x4910
0x000000000043f122 : adc al , 0x13 ; jmp rdx
...
```