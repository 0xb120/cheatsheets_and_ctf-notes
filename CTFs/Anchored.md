---
Category:
  - Mobile
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - Android
  - certificate-pinning-bypass
  - network_security_config.xml
  - patching-APK
  - reversing
---
>[!quote]
> A client asked me to check if I can intercept the https request and get the value of the secret parameter that is passed along with the user's email. The application is intended to run in a non-rooted device. Can you help me find a way to intercept this value in plain text.

# Set up

1. Install this application in a non-rooted device (i.e. In Android Studio AVD Manager select an image that includes (Google Play)).
    1. Used a Huawei P10 lite with Android 8.0
    2. Enabled debug USB and debug adb
2. Burp suite listening on every interface and VM on Bridged

# Information Gathering

![Application running on the device](../../zzz_res/attachments/Anchored%20c22062da48c04f39bd09de255194f34a.png)

Application running on the device

**AndroidManifest.xml**

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?><manifest xmlns:android="http://schemas.android.com/apk/res/android" android:compileSdkVersion="31" android:compileSdkVersionCodename="12" package="com.example.anchored" platformBuildVersionCode="31" platformBuildVersionName="12">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application android:allowBackup="true" android:appComponentFactory="androidx.core.app.CoreComponentFactory" android:icon="@mipmap/ic_launcher" android:label="@string/app_name" android:networkSecurityConfig="@xml/network_security_config" android:roundIcon="@mipmap/ic_launcher_round" android:supportsRtl="true" android:theme="@style/Theme.Anchored">
        <activity android:exported="true" android:name="com.example.anchored.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

The application perform do not trust user certificates but cannot run on rooted devices.

- Embed Frida gadget to APK? —> APK is obfuscated
- Edit/add **network_security_config.xml** in order to make the app trust user installed certificate

# Exploitation

Patched the **network_security_config.xml**

```xml
$ bat res/xml/network_security_config.xml 
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: res/xml/network_security_config.xml
       │ Size: 403 B
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ <?xml version="1.0" encoding="utf-8"?>
   2   │ <network-security-config>
   3   │     <domain-config cleartextTrafficPermitted="false">
   4   │         <domain includeSubdomains="true">anchored.com</domain>
   5   │         <trust-anchors>
   6   │         <certificates src="system" />
   7   │         <certificates src="user" overridePins="true" />
   8   │         <certificates src="@raw/certificate" />
   9   │         </trust-anchors>
  10   │     </domain-config>
  11   │ </network-security-config>
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Generated the key and signed the patched APK:

```jsx
$ keytool -genkey -v -keystore test.keystore -alias Test -keyalg RSA -keysize 1024 -sigalg SHA1withRSA -validity 10000
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter keystore password:
Re-enter new password: 
What is your first and last name?
  [Unknown]:  0xbro
What is the name of your organizational unit?
  [Unknown]:  maoutis
What is the name of your organization?
  [Unknown]:
What is the name of your City or Locality?
  [Unknown]:
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:
Is CN=0xbro, OU=maoutis, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Generating 1,024 bit RSA key pair and self-signed certificate (SHA1withRSA) with a validity of 10,000 days
        for: CN=0xbro, OU=maoutis, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing test.keystore]

Warning:
The generated certificate uses the SHA1withRSA signature algorithm which is considered a security risk. This algorithm will be disabled in a future update.
The generated certificate uses a 1024-bit RSA key which is considered a security risk. This key size will be disabled in a future update.

┌──(kali㉿kali)-[~/…/Anchored/apktool/Anchored/dist]
└─$ jarsigner -keystore test.keystore Anchored.apk -sigalg SHA1withRSA -digestalg SHA1 Test
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter Passphrase for keystore:
jar signed.

Warning:
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk. This algorithm will be disabled in a future update.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk. This algorithm will be disabled in a future update.
The RSA signing key has a keysize of 1024 which is considered a security risk. This key size will be disabled in a future update.

$ adb install Anchored.apk              
Performing Streamed Install
Success
```

![Anchored%20c22062da48c04f39bd09de255194f34a](../../zzz_res/attachments/Anchored%20c22062da48c04f39bd09de255194f34a%201.png)

# Flag

`HTB{UnTrUst3d_C3rT1f1C4T3s}`