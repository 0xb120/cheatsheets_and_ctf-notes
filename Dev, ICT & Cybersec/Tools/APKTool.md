---
Description: A tool for reverse engineering Android apk files
URL: https://ibotpeaches.github.io/Apktool/
---

>[!summary]
>A tool for reverse engineering 3rd party, closed, binary Android  apps. It can decode resources to nearly original form and rebuild them after making some modifications. It also makes working with an app easier because of the project like file structure and automation of some repetitive tasks like building apk, etc.

>[!warning]
>It is **NOT** intended for piracy and other non-legal uses. It could be used for localizing, adding some features or support for custom platforms, analyzing applications and much more.

# Decompile APK

```bash
$ apktool d Application.apk
```

# Rebuild files into APK

```bash
$ apktool b /folder
```

>[!warning]
>Sign the new APK with self-signed certificates in order to allow it to be installed on devices

```bash
# Generate key
$ keytool -genkey -v -keystore keys/test.keystore -alias Test -keyalg RSA -keysize 1024 -sigalg SHA1withRSA -validity 10000

# Sign APK
$ jarsigner -keystore keys/test.keystore dist/test.apk -sigalg SHA1withRSA -digestalg SHA1 Test
```