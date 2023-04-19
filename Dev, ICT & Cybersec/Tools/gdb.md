---
Description: GDB, the GNU Project debugger, allows you to see what is going on 'inside' another program while it executes -- or what another program was doing at the moment it crashed.
---

>[!info]
>GDB supports the following languages:
>- Ada
>- Assembly
>- C
>- C++
>- D
>- Fortran
>- Go
>- Objective-C
>- OpenCL
>- Modula-2
>- Pascal
>- Rust


## gdb plugins

Plugins dedicated to reversing and binary exploitation:

- [gdbpeda](https://github.com/longld/peda)
- [pwndbg](https://github.com/pwndbg/pwndbg)
- [gef](https://gef.readthedocs.io/en/master/)

## Custom settings

### Show disassembled code in Intel format

GDB naturally shows results in AT&T format. The disassembly syntax can be set to Intel typing runtime `set disassembly intel` or `set dis intel`

```bash
$ echo "set dis intel" > ~/.gdbinit
```

---

## gdb commands

### Start gdb

```bash
This is the GNU debugger.  Usage:

    gdb [options] [executable-file [core-file or process-id]]
    gdb [options] --args executable-file [inferior-arguments ...]

Selection of debuggee and its files:

  --args             Arguments after executable-file are passed to inferior
  --core=COREFILE    Analyze the core dump COREFILE.
  --exec=EXECFILE    Use EXECFILE as the executable.
  --pid=PID          Attach to running process PID.
  --directory=DIR    Search for source files in DIR.
  --se=FILE          Use FILE as symbol file and executable file.
  --symbols=SYMFILE  Read symbols from SYMFILE.
  --readnow          Fully read symbol files on first access.
  --readnever        Do not read symbol files.
  --write            Set writing into executable and core files.
```

Examples:

```bash
gdb name-of-executable
gdb name-of-executable --core name-of-core-file
gdb name-of-executable --pid=process-id
```

### Source code, disassembled code and memory mappings

| Action | Shortcut |
| --- | --- |
| List source code | l <br>l \<function\> <br>l \<line-number\> |
| Set number of lines listed using l | set listsize \<n\><br>show listsize |
| Disassemble from start to end | disas 0x\<start\> 0x\<end\> |
| Disassemble function | disas \<function_name\> |
| Displays the start and end position in object code for the current line in source | i line |
| Display position in object code for a specified line in source | i line \<number\> |
| Step/next assembly/processor instruction | si<br> ni |
| Information about the sections of the binary | info files |
| Information about the memory mappings | info proc mappings<br>vmmap |

### Breakpoints

| Action | Shortcut |
| --- | --- |
| At the beginning of a function | b \<function_name\> |
| On current line | b |
| At line n | b \<line_number\> |
| At address addr | b *\<address\> |
| n lines down/up from the current one | b +\<number_of_lines\><br>b -\<number_of_lines\> |
| Break if condition is true | b \<number\> if \<condition\> |
| List breakpoints | info break |
| Delete breakpoint number N | d \<breakpoint-number\> |
| Delete all breakpoints | clear |
| Delete function breakpoints | clear \<function\> |
| Delete breakpoint at number N | clear \<line-number\> |

### Program execution

| Action | Shortcut |
| --- | --- |
| Run until a breakpoint or an error | r |
| Continues running until a breakpoint or an error | c |
| Runs until the current function is finished | f |
| Runs the next line | s |
| Runs the next N lines of the program | s \<number\> |
| Runs the next line without step into functions | n |
| Runs until you get N lines in front of the current line | u \<number\> |

### Print values

| Action | Shortcut |
| --- | --- |
| Print variable var | p \<nome_var\> |
| Print N values for the array arr | p *\<arr\>@\<length\> |
| List registers in use | info registers |
| List threads in use | info threads |
| Examine the contents of memory | x 0x\<address\><br>x/\<nfu\> 0x\<address\> |
- *n*: number of display items to print
- *f*: specify the format for the output
    - `x` = hex
    - `d` = signed integer
    - `u` = unsigned integer
    - `o` = octal
    - `t` = binary
    - `c` = char
    - `f` = floating point number
    - `a` = hex address
- *u*: specify the size of the data unit (eg. byte, word, ...)
    - `q` = giant word
    - `w` = word
    - `h` = half word
    - `b` = signle byte

Example: `x/4dw var`

### Stack related commands

| Action | Shortcut |
| --- | --- |
| Print stack traces | bt |
| Print values of local variables | bt full |
| Show current stack frame (function where you are stopped) | frame
f \<number\> |
| Goes up N level in the stack | u
u \<number\> |
| Goes down N level in the stack | d
d \<number\> |
| List info about frame | info frame |
| Info arguments of selected frame, local variables and exception handlers | info args<br>info locals<br>info catch |

### Misc

| Action | Shortcut |
| --- | --- |
| Quits gdb | q |

## pwngdb specific commands
pwndbg documentation: [https://browserpwndbg.readthedocs.io/en/docs/](https://browserpwndbg.readthedocs.io/en/docs/)