---
Description: A runtime mobile exploration toolkit, powered by [Frida](https://www.frida.re/), built to help you assess the security posture of your mobile applications, without needing a jailbreak.
URL: https://github.com/sensepost/objection
---

![|700](../../zzz_res/attachments/objection.png)
![](../../zzz_res/attachments/objection2.png)

## Getting started

- https://github.com/sensepost/objection/wiki/Using-objection#getting-started-android-edition
- https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/frida-tutorial/objection-tutorial

```bash
$ objection --gadget "com.techuz.privatevault" explore

# Browse the file system
com.techuz.privatevault on (google: 10) [usb] # pwd
Current directory: /data/user/0/com.techuz.privatevault/files
com.techuz.privatevault on (google: 10) [usb] # ls
Type       Last Modified            Read  Write  Hidden  Size     Name
---------  -----------------------  ----  -----  ------  -------  ----------------------------------------------------------------------------------------------------
Directory  2023-06-26 21:45:55 GMT  True  True   True    4.0 KiB  .com.google.firebase.crashlytics
File       2023-06-14 20:58:12 GMT  True  True   False   0.0 B    generatefid.lock
File       2023-06-22 20:44:06 GMT  True  True   False   570.0 B  PersistedInstallation.W0RFRkFVTFRd+MTo1Nzk3OTA4MDI0ODM6YW5kcm9pZDozMGNkMjJjMTU4MDVjNDRlY2M5ZDQ2.json

Readable: True  Writable: True
com.techuz.privatevault on (google: 10) [usb] # env

Name                    Path
----------------------  -------------------------------------------------------------------
cacheDirectory          /data/user/0/com.techuz.privatevault/cache
codeCacheDirectory      /data/user/0/com.techuz.privatevault/code_cache
externalCacheDirectory  /storage/emulated/0/Android/data/com.techuz.privatevault/cache
filesDirectory          /data/user/0/com.techuz.privatevault/files
obbDir                  /storage/emulated/0/Android/obb/com.techuz.privatevault
packageCodePath         /data/app/com.techuz.privatevault-kpKamwMgXiwpOJfM11eD3g==/base.apk

# Download file
com.techuz.privatevault on (google: 10) [usb] # file download file.txt file.txt

# List activities
com.techuz.privatevault on (google: 10) [usb] # android hooking list activities

# Launch intents
com.opera.mini.native on (samsung: 6.0.1) [usb] # android intent launch_activity com.facebook.ads.AudienceNetworkActivity
Launching Activity: com.facebook.ads.AudienceNetworkActivity...
```