---
Category:
  - Misc
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
---
>[!quote]
> *We believe that there is an SSH Password inside password protected 'ZIP' folder. Can you crack the 'ZIP' folder and get the SSH password?*****


# Set up

- Download the provided file

# Information Gathering

- Zip file containing an SSH key

# The Bug

-

# Exploitation

```bash
┌──(maoutis㉿kali)-[~/Downloads]  
└─$ zip2john fsociety.zip  
Created directory: /home/maoutis/.john  
ver 2.0 fsociety.zip/sshcreds_datacenter.txt PKZIP Encr: cmplen=198, decmplen=729, crc=E126A116  
fsociety.zip/sshcreds_datacenter.txt:$pkzip2$1*1*2*0*c6*2d9*e126a116*0*35*8*c6*e126*8d9c*ee49af82113993a8062a3e309f126a6735d8fe7d6ca4382fdbd7aa6609b64411a43072212235835bb746967be74f4ea33014b695fe648799add3880671ae20caf3f854d73d6040dbb57f66db7328761e0cbecb85d5df465d4d4eabfee1fdbef6d9bbe2b6d86bc5bbdb2a30694181b7ec709c803022a7993fdca9234ebbabe54ec1dc118e49eff1faba92abd1eabcb1381d24139807604343caa2cf18359e3b7a594ed25d48805941deccf728f04fa4937a949c0c335344028a3f60eb74fc495e50f58ff8ad81*$/pkzip2$:sshcreds_datacenter.txt:fsociety.zip::fsociety.zip  
  
┌──(maoutis㉿kali)-[~/Downloads]  
└─$ zip2john fsociety.zip > hash.txt  
ver 2.0 fsociety.zip/sshcreds_datacenter.txt PKZIP Encr: cmplen=198, decmplen=729, crc=E126A116  

┌──(maoutis㉿kali)-[~/Downloads]  
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --fork=5 &nbsp; 
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Node numbers 1-5 of 5 (fork)  
Press 'q' or Ctrl-C to abort, almost any other key for status  
justdoit &nbsp; &nbsp; &nbsp; &nbsp; (fsociety.zip/sshcreds_datacenter.txt)  
3 1g 0:00:00:00 DONE (2021-09-22 09:53) 100.0g/s 70400p/s 70400c/s 70400C/s singing..herbert  
2 0g 0:00:00:00 DONE (2021-09-22 09:53) 0g/s 3630Kp/s 3630Kc/s 3630KC/s &nbsp;3117548331..*7¡Vamos!  
4 0g 0:00:00:00 DONE (2021-09-22 09:53) 0g/s 3186Kp/s 3186Kc/s 3186KC/s &nbsp;0557862091.ie168  
5 0g 0:00:00:00 DONE (2021-09-22 09:53) 0g/s 2987Kp/s 2987Kc/s 2987KC/s !!!!!v.abygurl69  
1 0g 0:00:00:00 DONE (2021-09-22 09:53) 0g/s 2897Kp/s 2897Kc/s 2897KC/s &nbsp; kaitlynn4.a6_123  
Waiting for 4 children to terminate  
Session completed  
  
┌──(maoutis㉿kali)-[~/Downloads]  
└─$ unzip fsociety.zip  
Archive: &nbsp;fsociety.zip  
[fsociety.zip] sshcreds_datacenter.txt password:  
&nbsp; inflating: sshcreds_datacenter.txt  
  
┌──(maoutis㉿kali)-[~/Downloads]  
└─$ cat sshcreds_datacenter.txt  
*****************************************************************************************  
Encrypted SSH credentials to access Blume ctOS :  
  
MDExMDEwMDEgMDExMDAxMTAgMDEwMTExMTEgMDExMTEwMDEgMDAxMTAwMDAgMDExMTAxMDEgMDEwMTExMTEgMDExMDAwMTEgMDEwMDAwMDAgMDExMDExMTAgMDEwMTExMTEgMDAxMDAxMDAgMDExMDExMDEgMDAxMTAwMTEgMDExMDExMDAgMDExMDExMDAgMDEwMTExMTEgMDExMTAxMTEgMDExMDEwMDAgMDEwMDAwMDAgMDExMTAxMDAgMDEwMTExMTEgMDExMTAxMDAgMDExMDEwMDAgMDAxMTAwMTEgMDEwMTExMTEgMDExMTAwMTAgMDAxMTAwMDAgMDExMDAwMTEgMDExMDEwMTEgMDEwMTExMTEgMDExMDEwMDEgMDExMTAwMTEgMDEwMTExMTEgMDExMDAwMTEgMDAxMTAwMDAgMDAxMTAwMDAgMDExMDEwMTEgMDExMDEwMDEgMDExMDExMTAgMDExMDAxMTE=  
  
***************************************************************************************** &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  
┌──(maoutis㉿kali)-[~/Downloads]  
└─$ echo 'MDExMDEwMDEgMDExMDAxMTAgMDEwMTExMTEgMDExMTEwMDEgMDAxMTAwMDAgMDExMTAxMDEgMDEwMTExMTEgMDExMDAwMTEgMDEwMDAwMDAgMDExMDExMTAgMDEwMTExMTEgMDAxMDAxMDAgMDExMDExMDEgMDAxMTAwMTEgMDExMDExMDAgMDExMDExMDAgMDEwMTExMTEgMDExMTAxMTEgMDExMDEwMDAgMDEwMDAwMDAgMDExMTAxMDAgMDEwMTExMTEgMDExMTAxMDAgMDExMDEwMDAgMDAxMTAwMTEgMDEwMTExMTEgMDExMTAwMTAgMDAxMTAwMDAgMDExMDAwMTEgMDExMDEwMTEgMDEwMTExMTEgMDExMDEwMDEgMDExMTAwMTEgMDEwMTExMTEgMDExMDAwMTEgMDAxMTAwMDAgMDAxMTAwMDAgMDExMDEwMTEgMDExMDEwMDEgMDExMDExMTAgMDExMDAxMTE=' | base64 -d  
01101001 01100110 01011111 01111001 00110000 01110101 01011111 01100011 01000000 01101110 01011111 00100100 01101101 00110011 01101100 01101100 01011111 01110111 01101000 01000000 01110100 01011111 01110100 01101000 00110011 01011111 01110010 00110000 01100011 01101011 01011111 01101001 01110011 01011111 01100011 00110000 00110000 01101011 01101001 01101110 01100111&nbsp;
```

![image.png](../../zzz_res/attachments/image.png)

# Flag

>[!success]
>`HTB{if_y0u_c@n_$m3ll_wh@t_th3_r0ck_is_c00king}`

