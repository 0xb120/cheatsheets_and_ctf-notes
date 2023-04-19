---
Description: It is a command-line program for displaying various information about object files on Unix-like operating systems. For instance, it can be used as a disassembler to view an executable in assembly form.
URL: https://man7.org/linux/man-pages/man1/objdump.1.html
---

```bash
objdump [-a|--archive-headers]
        [-b bfdname|--target=bfdname]
        [-C|--demangle[=style] ]
        [-d|--disassemble[=symbol]]
        [-D|--disassemble-all]
        [-z|--disassemble-zeroes]
        [-EB|-EL|--endian={big | little }]
        [-f|--file-headers]
        [-F|--file-offsets]
        [--file-start-context]
        [-g|--debugging]
        [-e|--debugging-tags]
        [-h|--section-headers|--headers]
        [-i|--info]
        [-j section|--section=section]
        [-l|--line-numbers]
        [-S|--source]
        [--source-comment[=text]]
        [-m machine|--architecture=machine]
        [-M options|--disassembler-options=options]
        [-p|--private-headers]
        [-P options|--private=options]
        [-r|--reloc]
        [-R|--dynamic-reloc]
        [-s|--full-contents]
        [-W[lLiaprmfFsoORtUuTgAckK]|
         --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,=frames-interp,=str,=str-offsets,=loc,=Ranges,=pubtypes,=trace_info,=trace_abbrev,=trace_aranges,=gdb_index,=addr,=cu_index,=links,=follow-links]]
        [--ctf=section]
        [-G|--stabs]
        [-t|--syms]
        [-T|--dynamic-syms]
        [-x|--all-headers]
        [-w|--wide]
        [--start-address=address]
        [--stop-address=address]
        [--no-addresses]
        [--prefix-addresses]
        [--[no-]show-raw-insn]
        [--adjust-vma=offset]
        [--dwarf-depth=n]
        [--dwarf-start=n]
        [--ctf-parent=section]
        [--no-recurse-limit|--recurse-limit]
        [--special-syms]
        [--prefix=prefix]
        [--prefix-strip=level]
        [--insn-width=width]
        [--visualize-jumps[=color|=extended-color|=off]
        [-V|--version]
        [-H|--help]
        objfile…

```

Display assembly code in Intel format:

```bash
$ objdump -D -M intel file.bin | grep main.: -A20
4004ed:	55                   	push   rbp
  4004ee:	48 89 e5             	mov    rbp,rsp
  4004f1:	c7 45 ec 00 00 00 00 	mov    DWORD PTR [rbp-0x14],0x0
  4004f8:	c7 45 f0 01 00 00 00 	mov    DWORD PTR [rbp-0x10],0x1
  4004ff:	c7 45 f4 02 00 00 00 	mov    DWORD PTR [rbp-0xc],0x2
  400506:	c7 45 f8 03 00 00 00 	mov    DWORD PTR [rbp-0x8],0x3
  40050d:	c7 45 fc 04 00 00 00 	mov    DWORD PTR [rbp-0x4],0x4
  400514:	c7 45 ec 00 00 00 00 	mov    DWORD PTR [rbp-0x14],0x0
  40051b:	eb 13                	jmp    400530 <main+0x43>
  40051d:	8b 05 15 0b 20 00    	mov    eax,DWORD PTR [rip+0x200b15]        # 601038 <globalA>
  400523:	83 e8 01             	sub    eax,0x1
  400526:	89 05 0c 0b 20 00    	mov    DWORD PTR [rip+0x200b0c],eax        # 601038 <globalA>
  40052c:	83 45 ec 01          	add    DWORD PTR [rbp-0x14],0x1
  400530:	8b 05 02 0b 20 00    	mov    eax,DWORD PTR [rip+0x200b02]        # 601038 <globalA>
  400536:	39 45 ec             	cmp    DWORD PTR [rbp-0x14],eax
  400539:	7c e2                	jl     40051d <main+0x30>
  40053b:	5d                   	pop    rbp
  40053c:	c3                   	ret    
  40053d:	0f 1f 00             	nop    DWORD PTR [rax]
```