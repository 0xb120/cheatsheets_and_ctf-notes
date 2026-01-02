---
Category:
  - Mobile
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [Android, hardcoded-credentials, patching-APK, reversing]
---
>[!quote]
> This app contains some unique keys. Can you get one?

# Set up

- Installed the app inside Genymotion
- Set up burpsuite as proxy

# Information Gathering

- Application does not perform HTTP requests when logging in
    
    ![APKey%20dc90602e985f4565afb8daf351283745](../../zzz_res/attachments/APKey%20dc90602e985f4565afb8daf351283745.png)
    

```bash
$ apktool d APKey.apk
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.5.0-dirty on APKey.apk
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
```

# The Bug

![Partially hardcoded MD5](../../zzz_res/attachments/APKey%20dc90602e985f4565afb8daf351283745%201.png)

Partially hardcoded MD5

# Exploitation

```bash
$ echo -n maoutis | md5sum                                              
6f2ae4978075eae54f9491744818d28d -

$ sed -i 's/a2a3d412e92d896134d9c9126d756f/6f2ae4978075eae54f9491744818d28d/' APKey/smali/com/example/apkey/MainActivity\$a.smali

$ wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.6.1.jar
$ chmod +x apktool_2.6.1.jar

$ java -jar apktool_2.6.1.jar b ./APKey -o test.apk
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
  [no]: yes

Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 10,000 days
        for: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing key.jks]

$ jarsigner -keystore key.jks test.apk HTB-alias
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter Passphrase for keystore: maoutis
jar signed.

Warning:
The signer's certificate is self-signed.
```

![Installed the patched app](../../zzz_res/attachments/APKey%20dc90602e985f4565afb8daf351283745%202.png)

Installed the patched app

![APKey%20dc90602e985f4565afb8daf351283745](../../zzz_res/attachments/APKey%20dc90602e985f4565afb8daf351283745%203.png)

# Flag

`HTB{m0r3_0bfusc4t1on_w0uld_n0t_hurt}`
