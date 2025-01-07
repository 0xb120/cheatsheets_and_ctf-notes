---
Description: Android Debug Bridge (adb) is a versatile command-line tool that lets you communicate with a device. The adb command facilitates a variety of device actions, such as installing and debugging apps, and it provides access to a Unix shell that you can use to run a variety of commands on a device.
URL: https://developer.android.com/studio/releases/platform-tools
---

>[!info]
>`adb` is included in the Android SDK Platform-Tools package. You can download this package with the [android_sdk](android_sdk.md), which installs it at `android_sdk/platform-tools/`. Or if you want the standalone Android SDK Platform-Tools package, you can [download it here](https://developer.android.com/studio/releases/platform-tools) [^1].

[^1]: https://developer.android.com/studio/command-line/adb

- adb source code: https://android.googlesource.com/platform/packages/modules/adb/
- frameworks base: https://android.googlesource.com/platform/frameworks/base/
- Other commands: https://blog.testproject.io/2021/08/10/useful-adb-commands-for-android-testing/

# General commands

## List connected devices

```bash
$ adb devices
List of devices attached
emulator-5554 device
```

## Start and stop adb server

```bash
$ adb start-server
* daemon not running. starting it now on port 5037 *
* daemon started successfully *
$ adb kill-server
```

## Connect via TCP-IP / USB

```bash
$ adb tcpip <port>
$ adb connect <IP> <port>
$ adb usb
```

## Spawn a shell

```bash
$ adb shell

# Specify which device adb should interact with
$ adb -s "device_id" shell

# Use the USB device
$ adb -d shell 
```

## List and interact with installed packages

```bash
$ adb shell pm list packages
$ adb shell pm list packages -3 # List only third party packages
$ adb shell pm path <package_name> # Get the full path of the APK

# Clear the application info without removing the app
$ adb shell pm clear <package_name>
```

## Inspect device log

```bash
$ adb logcat
$ adb logcat -v <log_format>
$ adb logcat -d -f log.txt # -d just dump the file; -f save the log inside a file insted printing to stdout
$ adb logcat *:E # dump only Error priority or higher
$ adb logcat MyTag:I *:S # prints to the output log messages with the tag MyTag and priority level Info or higher. The *:S at the end will exclude the log from other tags with any priority
```

- **V**: Verbose (lowest priority)
- **D**: Debug
- **I**: Info
- **W**: Warning
- **E**: Error
- **F**: Fatal
- **S**: Silent (highest priority. Nothing is printed)

## Push and pull files from the device

```bash
$ adb push <file> <destination> # upload
$ adb pull <file> <destination> # download
```

## Battery related commands

```bash
# Set battery to 1%
adb shell dumpsys battery set level 1

# connect/disconnect an AC charger
adb shell dumpsys battery set ac 1

# connect/disconnect an USB charger
adb shell dumpsys battery set usb 1

# Reset to standard
adb shell dumpsys battery reset
```

## Take photos and videos

```bash
adb shell screenrecord /sdcard/Movies/video.mp4
adb shell screencap -p /sdcard/Pictures/screenshot.png
```

---

# Static analysis

## Extract APKs without root privileges

```bash
$ adb shell pm list packages  
$ adb shell pm path <package_name>
$ adb pull <APK_path>
```

## Extract interesting information from packages

```bash
# Inspect memory consumption by packages
$ adb shell dumpsys meminfo

# List package information, including manifest, providers, etc.
$ adb shell dumpsys package name.of.package.class
```

## Open SQLite db

```bash
$ sqlite3 /path/to/file
sqlite> .tables
Files             SubDirectory      android_metadata
Notes             Types
sqlite> select * from Notes;
```

---

# Dynamic analysis


## Push burpsuite certificate inside the system certificate store

### Android >= 7 & Android <14

>[!tldr] TL.DR
>- [Install System CA Certificate on Android Emulator](https://docs.mitmproxy.org/stable/howto-install-system-trusted-ca-android/)
>- [Insert certificate into system certificate store by dylanninin](https://gist.github.com/pwlin/8a0d01e6428b7a96e2eb?permalink_comment_id=3927718#gistcomment-3927718)
>- [ADB interception](https://httptoolkit.com/docs/guides/android/#adb-interception)

```bash
# convert burpsuite certificate from der to cer/pem/crt
openssl x509 -inform der -in burp-free.der -out burp-free.cer

# extract the hash to be used for the certificate name
openssl x509 -inform PEM -subject_hash_old -in burp-free.cer | head -1
9a5ba575
cp burp-free.cer 9a5ba575.0

# install the certificate inside the system certificate store
.\emulator.exe -avd <avd_name_here> -writable-system
.\adb.exe root
.\adb.exe disable-verity
.\adb.exe reboot
.\adb.exe root
.\adb.exe remount
.\adb.exe push 9a5ba575.0 /system/etc/security/cacerts
.\adb.exe shell chmod 644 /system/etc/security/cacerts/9a5ba575.0
.\adb.exe reboot
```

If the method above not worked and the emulator stuck on reboot [^solution] :

[^solution]: https://stackoverflow.com/questions/63875910/android-emulator-stuck-on-reboot-after-adb-disable-verity-or-adb-remount
```bash
.\emulator.exe -avd <avd_name_here> -writable-system
.\adb.exe root
.\adb.exe shell avbctl disable-verification
.\adb.exe reboot
.\adb.exe root
.\adb.exe remount
.\adb.exe push 9a5ba575.0 /system/etc/security/cacerts
.\adb.exe shell chmod 644 /system/etc/security/cacerts/9a5ba575.0
.\adb.exe reboot
```

### Android >14 [^certs-android-14]

[^certs-android-14]: [httptoolkit.com - New Ways to Inject System CA Certificates in Android 14](../../Readwise/Articles/httptoolkit.com%20-%20New%20Ways%20to%20Inject%20System%20CA%20Certificates%20in%20Android%2014.md)

>[!warning]
>Both the techniques are **temporary**! The injected certificates only last until the next reboot.

In Android 14, system-trusted CA certificates will generally live in `/apex/com.android.conscrypt/cacerts`, and all of `/apex` is immutable. This means that APEX cacerts path cannot be remounted as rewritable - remounts simply fail. 

The reason is that `/apex` mount is [explicitly mounted](https://cs.android.com/android/platform/superproject/main/+/main:system/core/init/mount_namespace.cpp;l=97;drc=566c65239f1cf3fcb0d8745715e5ef1083d4bd3a) with PRIVATE propagation, so that all changes to mounts inside the `/apex` path are never shared between processes. However we are root, so using `nsenter`, we can run code in other namespaces!

Bind-mounting through NSEnter:
```bash
mount -t tmpfs tmpfs /system/etc/security/cacerts

# place the CA certificates you're interested in into this directory and set permissions & SELinux labels appropriately

nsenter --mount=/proc/$ZYGOTE_PID/ns/mnt -- \
    /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts
    
nsenter --mount=/proc/$APP_PID/ns/mnt -- \
    /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts
```

Automated script:
```bash
# Create a separate temp directory, to hold the current certificates
# Otherwise, when we add the mount we can't read the current certs anymore.
mkdir -p -m 700 /data/local/tmp/tmp-ca-copy

# Copy out the existing certificates
cp /apex/com.android.conscrypt/cacerts/* /data/local/tmp/tmp-ca-copy/

# Create the in-memory mount on top of the system certs folder
mount -t tmpfs tmpfs /system/etc/security/cacerts

# Copy the existing certs back into the tmpfs, so we keep trusting them
mv /data/local/tmp/tmp-ca-copy/* /system/etc/security/cacerts/

# Copy our new cert in, so we trust that too
mv $CERTIFICATE_PATH /system/etc/security/cacerts/

# Update the perms & selinux context labels
chown root:root /system/etc/security/cacerts/*
chmod 644 /system/etc/security/cacerts/*
chcon u:object_r:system_file:s0 /system/etc/security/cacerts/*

# Deal with the APEX overrides, which need injecting into each namespace:

# First we get the Zygote process(es), which launch each app
ZYGOTE_PID=$(pidof zygote || true)
ZYGOTE64_PID=$(pidof zygote64 || true)
# N.b. some devices appear to have both!

# Apps inherit the Zygote's mounts at startup, so we inject here to ensure
# all newly started apps will see these certs straight away:
for Z_PID in "$ZYGOTE_PID $ZYGOTE64_PID"; do
    # We use 'echo' below to trim spaces
    nsenter --mount=/proc/$(echo $Z_PID)/ns/mnt -- \
        /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts
done

# Then we inject the mount into all already running apps, so they
# too see these CA certs immediately:

# Get the PID of every process whose parent is one of the Zygotes:
APP_PIDS=$(
    echo "$ZYGOTE_PID $ZYGOTE64_PID" | \
    xargs -n1 ps -o 'PID' -P | \
    grep -v PID
)

# Inject into the mount namespace of each of those apps:
for PID in $APP_PIDS; do
    nsenter --mount=/proc/$PID/ns/mnt -- \
        /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts &
done
wait # Launched in parallel - wait for completion here

echo "System certificate injected"
```

Recursively remounting mountpoints: [^g1a55er]

[^g1a55er]: [Android 14 Still Allows Modification of System Certificates, g1a55er](https://www.g1a55er.net/Android-14-Still-Allows-Modification-of-System-Certificates)

- You can remount `/apex` manually, removing the PRIVATE propagation and making it writable (ironically, it seems that entirely removing private propagation _does_ propagate everywhere)
- You copy out the entire contents of `/apex/com.android.conscrypt` elsewhere
- Then you unmount `/apex/com.android.conscrypt` entirely - removing the read-only mount that immutably provides this module
- Then you copy the contents back, so it lives into the `/apex` mount directly, where it can be modified (you need to do this quickly, as [apparently](https://infosec.exchange/@g1a55er/111069489513139531) you can see crashes otherwise)
- This should take effect immediately, but they recommend killing `system_server` (restarting all apps) to get everything back into a consistent state

## Activity interaction

```bash
# start an activity
adb shell am start -n com.example.demo/com.example.test.MainActivity
```

## Content provider interaction

```bash
# List all the exported providers for any application
adb shell dumpsys package providers

# Interact with a specific content provider
adb shell content read --uri content://com.techuz.privatevault/data/data/com.techuz.privatevault/shared_prefs/PV.xml
adb shell content delete --uri content://com.techuz.privatevault/data/data/com.techuz.privatevault/shared_prefs/PV.xml
```

## Broadcast receiver interaction

```bash
adb shell am broadcast -a com.whereismywifeserver.intent.TEST --es sms_body "test from adb"
```

## Service interaction
```xml
<service  android:name="com.yourpackage.SomeService">
     <intent-filter>
            <action android:name="com.yourpackage.action.name.SHOW_TOAST" />
     </intent-filter>
 </service>
```

```bash
adb shell am startservice -a com.yourpackage.action.name.SHOW_TOAST -e text "i did it" # for the @Override of onStart()
```

## Stop application

```bash
adb shell am force-stop
```

## Simulate input, tap, scroll and swipe

```bash
adb shell input text "this\ is\ some\ string"
adb shell input tap 500 400
adb shell input swipe 500 1000 500 100 # Perform a swipe gesture from the center to top
adb shell input swipe 500 1000 500 100 5000 # Perform the same gesture slowly. Takes 5 seconds

# Press Home button
$ adb shell input keyevent KEYCODE_HOME

# Press Camera button
$ adb shell input keyevent KEYCODE_CAMERA

# Press Back button
$ adb shell input keyevent KEYCODE_BACK

# Press Headset button
$ adb shell input keyevent KEYCODE_HEADSETHOOK
```