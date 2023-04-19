---
Category: Mobile
Difficulty: Easy
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags: Android, insecure-password-change
---
>[!quote]
>*A client asked me to perform security assessment on this password management application. Can you help me?*


# Set up

```bash
$ ls -al                
total 5392
drwxr-xr-x 2 kali kali    4096 Feb 27 14:17 .
drwxr-xr-x 3 kali kali    4096 Feb 27 14:15 ..
-rw-r--r-- 1 kali kali 3063982 Nov 26 19:21 Manager.apk
-rwxrwxrwx 1 kali kali 2440533 Feb 27 14:14 Manager.zip
-rw-r--r-- 1 kali kali     231 Nov 29 05:40 README.txt
                                                                                                                                                                                                                                            
$ cat README.txt 
1. Install this application in an API Level 29 or earlier (i.e. Android 10.0 (Google APIs)).

2. In order to connect to the server when first running the application, insert the IP and PORT that you are provided in the description.
```

![Installed the APK on Genymotion](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f.png)

Installed the APK on Genymotion

Set up a proxy in order to intercept the traffic with burpsuite

# Information Gathering

```bash
$ apktool d Manager.apk
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.5.0-dirty on Manager.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/kali/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/Challenge/Mobile/Manager]
└─$ ls Manager
AndroidManifest.xml  apktool.yml  original  res  smali
```

![Login page](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%201.png)

Login page

![User enumeration —> No, default error message](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%202.png)

User enumeration —> No, default error message

![Registered a user](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%203.png)

Registered a user

![Password update](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%204.png)

Password update

# The Bug

![Password update does not require current password and does not check user authorization (no session management)](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%205.png)

Password update does not require current password and does not check user authorization (no session management)

# Exploitation

![Logged in with the took-over account](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%206.png)

Logged in with the took-over account

![Untitled](../../zzz_res/attachments/Manager%2054abc505948d4164a3cd292d32fb056f%207.png)

# Flag

>[!success]
>`HTB{b4d_p@ss_m4n@g3m3nT_@pp}`
