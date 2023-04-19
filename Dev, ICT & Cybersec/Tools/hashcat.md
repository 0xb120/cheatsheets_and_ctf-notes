---
Description: The world’s fastest and most advanced password recovery tool.
URL: https://hashcat.net/hashcat/
---

> [!info]
> Hashcat leverage the power of both the CPU and the GPU to reach incredible password cracking speeds.

```bash
kali@kali:~$ hashcat -m 100 -a0 -o hugo.pwd hugo.hash /usr/share/wordlists/rockyou.txt --force
...
Approaching final keyspace - workload adjusted.  

faca404fd5c0a31cf1897b823c695c85cffeb98d:Password120
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: SHA1
Hash.Target......: faca404fd5c0a31cf1897b823c695c85cffeb98d
Time.Started.....: Thu Jun  4 22:21:25 2020 (4 secs)
Time.Estimated...: Thu Jun  4 22:21:29 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3385.4 kH/s (0.15ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 14344386/14344386 (100.00%)
Rejected.........: 0/14344386 (0.00%)
Restore.Point....: 14344192/14344386 (100.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....:  kristenanne -> Password120
```

## Specifies username and delimiter

```bash
kali@kali:~$ hashcat -m1600 xampp-password /usr/share/wordlists/rockyou.txt --username --separator ":"
```

## Cracking MySql hash

```bash
$ hashcat -m 300 mysql_hash /usr/share/seclists/Passwords/2020-200_most_used_passwords.txt
```

## Kerberoasting

```bash
hashcat -m13100 TGS /usr/share/wordlists/rockyou.txt --force
```

## JWT HMAC

```
hashcat -m 16500 -a 0 jwt.txt .\wordlists\rockyou.txt
```

## Show already cracked password

```bash
┌──(kali㉿kali)-[~/…/HTB/box/Pit/loot]
└─$ hashcat users.hash --show --separator ":" --username
michelle:2345f10bb948c5665ef91f6773b3e455:michelle
```