---
Description: John the Ripper is an Open Source password security auditing and password recovery tool available for many operating systems.
URL: https://www.openwall.com/john/
---

## Dictionary generation

`/etc/john/john.conf` can be modify in order to add [custom rules](https://www.openwall.com/john/doc/RULES.shtml) to modify existing wordlists and create newer one.

```bash
$ root@kali:~$ nano /etc/john/john.conf

...
# Wordlist mode rules
[List.Rules:Wordlist]
# Try words as they are
:
# Lowercase every pure alphanumeric word
-c >3 !?X l Q
# Capitalize every pure alphanumeric word
-c (?a >2 !?X c Q
# Lowercase and pluralize pure alphabetic words
...
# Try the second half of split passwords
-s x_
-s-c x_ M l Q
...
[List.Rules:Custom]
:
# Add one number
$[0-9]
# Add one number and a sybol
$[0-9]$[$%^&*()\\-_+=|\<>\[\]{}#@/~]
# Add two number
$[0-9]$[0-9]
# Add two number and a symbol
$[0-9]$[0-9]$[$%^&*()\\-_+=|\<>\[\]{}#@/~]
...

$ john root.passwd --wordlist=base_passwords --rules=Custom --fork=5       # Runtime
$ john --wordlist=megacorpone.txt --rules --stdout > new_megacorpone.txt   # Create wordlist
```

`$[0-9]`: append a digit from 0 to 9
`$[\$\%\^\&\*\(\)\-\_\+\=\|\\\<\>\[\]\{\}\#\@\/\~]$[\$\%\^\&\*\(\)\-\_\+\=\|\\\<\>\[\]\{\}\#\@\/\~]`: append a special character
`cAz`: capitalize from a to Z

More rules [here](https://www.openwall.com/john/doc/RULES.shtml)

## Password cracking examples

### Cracking NTLM hashes

```bash
kali@kali:~$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=NT
Using default input encoding: UTF-8
Rules/masks using ISO-8859-1
Loaded 2 password hashes with no different salts (NT [MD4 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
```

### Cracking Shadow Files

```bash
┌──(kali㉿kali)-[~/…/ntwk/student/72/loot]
└─$ unshadow passwd.split shadow.split > unshadowed
                                                                                                                                                                                                                  
┌──(kali㉿kali)-[~/…/ntwk/student/72/loot]
└─$ john unshadowed --wordlist=/home/kali/Documents/lab/ntwk/custom_lists/creds.list --fork=5
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Node numbers 1-5 of 5 (fork)
4: Warning: Only 1 candidate left, minimum 2 needed for performance.
5: Warning: Only 1 candidate left, minimum 2 needed for performance.
4 0g 0:00:00:00 DONE (2021-02-23 15:59) 0g/s 100.0p/s 200.0c/s 200.0C/s j957bjc6qczq2gpm
Press 'q' or Ctrl-C to abort, almost any other key for status
3 0g 0:00:00:00 DONE (2021-02-23 15:59) 0g/s 100.0p/s 200.0c/s 200.0C/s hill..tanya4life
1 0g 0:00:00:00 DONE (2021-02-23 15:59) 0g/s 200.0p/s 400.0c/s 400.0C/s zaq1xsw2cde3..aliceishere
Waiting for 4 children to terminate
5 0g 0:00:00:00 DONE (2021-02-23 15:59) 0g/s 100.0p/s 200.0c/s 200.0C/s gonz
QUHqhUPRKXMo4m7k (ryuu)
2 1g 0:00:00:00 DONE (2021-02-23 15:59) 100.0g/s 200.0p/s 400.0c/s 400.0C/s QUHqhUPRKXMo4m7k..3v1lp@ss
Session completed
                                                                                                                                                                                                                  
┌──(kali㉿kali)-[~/…/ntwk/student/72/loot]
└─$ john unshadowed --show                                                                   
ryuu:QUHqhUPRKXMo4m7k:1010:1010::/home/ryuu:/bin/rbash

1 password hash cracked, 1 left
```

## Cracking HMAC JWT

```
john jwt.txt --wordlist=wordlists.txt --format=HMAC-SHA256
```

### Applying runtime rules

```bash
kali@kali:~$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=NT --rules
Using default input encoding: UTF-8
Rules/masks using ISO-8859-1
Loaded 2 password hashes with no different salts (NT [MD4 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
```

### Fork and distribute the work

```bash
kali@kali:~$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=NT --rules --fork=8 --node=1-2/4
Using default input encoding: UTF-8
Rules/masks using ISO-8859-1
Loaded 2 password hashes with no different salts (NT [MD4 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
```

### Show results

```bash
kali@kali:~$ john unshadowed --show                                                                   
gibson:zaq1xsw2cde3:1000:1000:gibson,,,:/home/gibson:/bin/bash

1 password hash cracked, 1 left
```

## External scripts

- `pdf2john.pl`: extracts the hash inside a PDF file in order to auditing it with john or [hashcat](hashcat.md)
- `ssh2john`: extracts the hash inside a SSH key file in order to auditing it with john or [hashcat](hashcat.md)
- `kirbi2john`: transform a Kerberos TGS in a suitable hash for john