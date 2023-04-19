---
Category: Mobile
Difficulty: Easy
Platform: HackTheBox
Retired: false
Status: 3. Complete
Tags: Android, React, hardcoded-key
---
>[!quote]
>*Some web developers wrote this fancy new app! It's really cool, isn't it?*


# Set up

```bash
$ unzip dont_overreact.zip
Archive:  dont_overreact.zip
[dont_overreact.zip] app-release.apk password:
  inflating: app-release.apk

$ ls
app-release.apk  dont_overreact.zip
```

# Information Gathering

```bash
$ apktool d app-release.apk
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.5.0-dirty on app-release.apk
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

$ ls
app-release  app-release.apk  dont_overreact.zip

$ ls app-release
AndroidManifest.xml  apktool.yml  assets  lib  original  res  smali  unknown
```

# The Bug

![Hardcoded debug information](../../zzz_res/attachments/Don't%20Overreact%2045c55b99e9464e19900d439f380d360a.png)

>Hardcoded debug information

# Exploitation

```bash
$ echo 'SFRCezIzbTQxbl9jNDFtXzRuZF9kMG43XzB2MzIyMzRjN30=' | base64 -d
HTB{23m41n_c41m_4nd_d0n7_0v32234c7}
```

# Flag

>[!success]
>`HTB{23m41n_c41m_4nd_d0n7_0v32234c7}`


