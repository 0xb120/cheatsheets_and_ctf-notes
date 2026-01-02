---
Platform: X-MAS
Difficulty: Easy
Category:
  - Misc
Status: 3. Complete
tags: [cracking-kdbx-files, steganography, blowfish, image-manipulation]
---

>[!quote]
>We've suffered a breach in our database and some of our files have been stolen by the Grinch. We managed to get some of them back, but we need your help in recovering the lost data, most especially because we think some of them have been altered in some way.
>
>PS: Start with the image then continue with the zip, Keepass file. Not your typical kind of steganography. :). ”Think like an attacker”  
>
>Password: Christmas

# Set up

Files: https://drive.google.com/file/d/1EVWwmzYMvy-w6LEl_SXgWHB5oKidPUcg/view

```bash
maoutis@Dell-XPS13:/mnt/c/Users/Bro/Documents/VM-Shared/manipulated$ ls -al
total 168
drwxrwxrwx 1 maoutis maoutis  4096 Dec 18 23:36  .
drwxrwxrwx 1 maoutis maoutis  4096 Dec 20 10:40  ..
-rwxrwxrwx 1 maoutis maoutis  1550 Dec 17 23:26  Flag.kdbx
-rwxrwxrwx 1 maoutis maoutis 46926 Dec 17 23:53  X-MAS.jpg
-rwxrwxrwx 1 maoutis maoutis   215 Dec 17 23:56  flag.zip
-rwxrwxrwx 1 maoutis maoutis 36999 Dec 18 16:24 'manipulated.zip'
```

X-MAS.jpg:
![](../../zzz_res/attachments/X-MAS.jpg)

# Cracked the kdbx file

```bash
┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ keepass2john Flag.kdbx 
Flag:$keepass$*2*100000*0*570238c5df658d5585014b1578f2a73812032cb599ce7bfe3961ea931df7bd31*76d389a71f5b064b6bd716c93fbe668232f665fe32eb5add3db005757aeea87d*befceb83b8fe811dfedb176ac23d8e27*3c348e19ba7b628e98e3e03a134838ae3d29ae59e8b8c6b5262e9097b154534f*76a25a61fae8d29ad0b935eb8cdce2e360975e6db23d3bb15e2e94c7700f7424

┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ keepass2john Flag.kdbx > kepass.hash

┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ john kepass.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 100000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dracula          (Flag)
1g 0:00:00:54 DONE (2022-12-22 04:03) 0.01818g/s 64.02p/s 64.02c/s 64.02C/s Password1..dracula
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ hashcat -m 13400 kepass2.hash /usr/share/wordlists/rockyou.txt --show
$keepass$*2*100000*0*570238c5df658d5585014b1578f2a73812032cb599ce7bfe3961ea931df7bd31*76d389a71f5b064b6bd716c93fbe668232f665fe32eb5add3db005757aeea87d*befceb83b8fe811dfedb176ac23d8e27*3c348e19ba7b628e98e3e03a134838ae3d29ae59e8b8c6b5262e9097b154534f*76a25a61fae8d29ad0b935eb8cdce2e360975e6db23d3bb15e2e94c7700f7424:dracula
```

>[!important] Password
>dracula

![](../../zzz_res/attachments/Pasted%20image%2020221222100718.png)

## Blowfish

>[!quote]
>We managed to extract this data but I can't figure out what it is. The attacker said it should be something easy. We only know that he likes some good cooked fish.
>Q2hyNGlzdG1hc0pveQ==
>N+ke3xIGF/h//tiT4SxIECOxGG7moZui0dccxtqmUg0=

![](../../zzz_res/attachments/Pasted%20image%2020221222100831.png)

>[!important]
>URL: https://pastebin.com/CPzeYJmb
>Blowfish password: Chr4istmasJoy

# Find the secret information inside the image

```bash
$ exiftool X-MAS.jpg
...
Image Size                      : 1000x896
Megapixels                      : 0.896
```

Extended height of the image and extracted the flag:
![](../../zzz_res/attachments/Pasted%20image%2020221222101403.png)

>[!important] Password
>RUDOLF_1S_4_N4UGHTY_BOY

## Extracted passwd from flag.zip

```bash
┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ unzip flag.zip
Archive:  flag.zip
[flag.zip] passwd.txt password:
 extracting: passwd.txt

┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ cat passwd.txt
ZlZlZVI3N0p1YQ==

┌──(kali㉿kali)-[~/CTFs/X-MAS/manipulated]
└─$ cat passwd.txt| base64 -d
fVeeR77Jua 
```

# Accessed the private pastebin

![](../../zzz_res/attachments/Pasted%20image%2020221222101717.png)
![](../../zzz_res/attachments/Pasted%20image%2020221222101737.png)