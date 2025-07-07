---
Category:
  - Mobile
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - Android
  - hardcoded-credentials
  - patching-APK
  - reversing
---
>[!quote]
> Can you get the ticket without the VIP code?

# Set up

- Install this application in an API Level 29 or earlier (i.e. Android 10.0 (Google APIs)).

# Information Gathering

![Main Activity](../../zzz_res/attachments/APKrypt%206b142c6e385f4024b40640d93082375e.png)

Main Activity

```bash
$ apktool d APKrypt.apk
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.5.0-dirty on APKrypt.apk
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

$ ls APKrypt
AndroidManifest.xml  apktool.yml  original  res  smali
```

# The Bug

![APKrypt%206b142c6e385f4024b40640d93082375e](../../zzz_res/attachments/APKrypt%206b142c6e385f4024b40640d93082375e%201.png)

# Exploitation

```bash
$ echo -n maoutis | md5sum
6f2ae4978075eae54f9491744818d28d  -

$ grep -ri "735c3628699822c4c1c09219f317a8e9"
smali/com/example/apkrypt/MainActivity$1.smali:    const-string v0, "735c3628699822c4c1c09219f317a8e9"

$ sed -i 's/735c3628699822c4c1c09219f317a8e9/6f2ae4978075eae54f9491744818d28d/' smali/com/example/apkrypt/MainActivity\$1.smali

$ java -jar /opt/Android/apktool_2.6.1.jar b ./APKrypt
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.6.1
I: Checking whether sources has changed...
I: Checking whether resources has changed...
I: Building resources...
I: Building apk file...
I: Copying unknown files/dir...
I: Built apk...

$ keytool -genkey -v -keystore key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias HTB-alias
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:
What is the name of your organizational unit?
  [Unknown]:
What is the name of your organization?
  [Unknown]:
What is the name of your City or Locality?
  [Unknown]:
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:
Is CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 10,000 days
        for: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing key.jks]

$ jarsigner -keystore key.jks APKrypt.apk HTB-alias
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter Passphrase for keystore:
jar signed.

Warning:
The signer's certificate is self-signed.

$ sudo cp APKrypt.apk /mnt/hgfs/VM-Shared/HTB/APKrypt-patched.apk
```

![APKrypt%206b142c6e385f4024b40640d93082375e](../../zzz_res/attachments/APKrypt%206b142c6e385f4024b40640d93082375e%202.png)

# Flag

`HTB{3nj0y_y0ur_v1p_subscr1pt1on}`

# YT Video

<iframe width="660" height="415" src="https://www.youtube.com/embed/gPPi3v8iRog" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


