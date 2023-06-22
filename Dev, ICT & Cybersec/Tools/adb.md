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
```

## List all installed packages

```bash
$ adb shell pm list packages
```

## Inspect device log

```bash
$ adb logcat
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

- https://docs.mitmproxy.org/stable/howto-install-system-trusted-ca-android/
- https://gist.github.com/pwlin/8a0d01e6428b7a96e2eb?permalink_comment_id=3927718#gistcomment-3927718
- https://httptoolkit.com/docs/guides/android/#adb-interception

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