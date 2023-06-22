# Available sdk 

>[!abstract] Docs
>https://developer.android.com/studio/command-line

```bash
PS C:\Users\mbrol\Desktop\android_tools\android_sdk\cmdline-tools\latest\bin> ls

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        24/03/2023     20:18           2297 apkanalyzer.bat
-a----        24/03/2023     20:18           2288 avdmanager.bat
-a----        24/03/2023     20:18           2242 lint.bat
-a----        24/03/2023     20:18           2222 profgen.bat
-a----        24/03/2023     20:18           2222 retrace.bat
-a----        24/03/2023     20:18           2285 screenshot2.bat
-a----        24/03/2023     20:18           2295 sdkmanager.bat
```

## sdkmanager

>[!tldr] Docs
>https://developer.android.com/studio/command-line/sdkmanager

The `sdkmanager` is a command-line tool that lets you view, install, update, and uninstall packages for the Android SDK. 

```bash
# list all the available sdk
 .\sdkmanager.bat --list
...
system-images;android-29;google_apis_playstore;x86                                       | 8            | Google Play Intel x86 Atom System Image
system-images;android-29;google_apis_playstore;x86_64                                    | 8            | Google Play Intel x86_64 Atom System Image
system-images;android-30;android-tv;x86                                                  | 4            | Android TV Intel x86 Atom System Image
system-images;android-30;android-wear-cn;arm64-v8a                                       | 10           | China version of Wear OS 3 - Preview ARM 64 v8a System Image
...

# install a specific sdk
.\sdkmanager.bat --install "system-images;android-29;google_apis;x86"

# list installed sdk 
.\sdkmanager.bat --list_installed
```

## avdmanager

>[!tldr] Doc
>https://developer.android.com/studio/command-line/avdmanager

The `avdmanager` is a command-line tool that lets you create and manage Android Virtual Devices (AVDs) from the command line. An AVD lets you define the characteristics of an Android handset, Wear OS watch, or Android TV device that you want to simulate in the Android Emulator.

```bash
# list all possible emulators
.\avdmanager.bat list devices

# create an emulator (C:\Users\name\.android\avd\)
.\avdmanager.bat --verbose create avd --force --name "generic_api29_google_apis_emulator" --package "system-images;android-29;google_apis;x86" --tag "google_apis" --abi "x86"
.\avdmanager.bat create avd -n "generic_api29_google_apis_emulator" -k "system-images;android-29;google_apis;x86"

# list installed avd
.\avdmanager.bat list avd
Available Android Virtual Devices:
    Name: generic_api29_google_apis_emulator
    Path: C:\Users\mbrol\.android\avd\generic_api29_google_apis_emulator.avd
  Target: Google APIs (Google Inc.)
          Based on: Android 10.0 (Q) Tag/ABI: google_apis/x86
  Sdcard: 512 MB

# delete emulator
 .\avdmanager.bat delete avd -n generic_api29_google_apis_emulator

# rename emulator
.\avdmanager.bat move avd -n generic_28 -r android-9_api-28
```

## Emulator

>[!warning]
>Manually creates all the missing folders otherwise `emulator.exe` will not work!

>The folder structure is the following one:
```bash
> Directory: C:\Users\mbrol\Desktop\android_tools\android_sdk


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        30/03/2023     11:22                .temp
d-----        30/03/2023     00:13                cmdline-tools
d-----        30/03/2023     00:55                emulator
d-----        30/03/2023     11:18                licenses
d-----        30/03/2023     00:54                patcher
d-----        30/03/2023     10:40                platform-tools
d-----        30/03/2023     10:29                platforms
d-----        30/03/2023     11:18                system-images
-a----        30/03/2023     11:24             16 .knownPackages
```

```bash
# list available emulators
PS C:\Users\mbrol\Desktop\android_tools\android_sdk\emulator> .\emulator.exe -list-avds
generic_api29_google_apis_emulator

# start a device
.\emulator.exe -avd generic_28

# start a device in writable mode with a bigger GUI
.\emulator.exe -avd generic_28 -writable-system -scale 0.6
```

Enable hardware buttons [^avd-hw] :

[^avd-hw]: [https://stackoverflow.com/questions/15100251/avd-hardware-buttons-not-enabled](https://stackoverflow.com/questions/15100251/avd-hardware-buttons-not-enabled)

```txt
$ edit ~/.android/avd/<emulator name>/config.ini
...
hw.keyboard=yes
...
hw.mainKeys=yes
...
```

## Port forwarding and network management

>[!tldr] RTFM
>https://developer.android.com/studio/run/emulator-networking

```bash
# Auth credential
more C:\Users\mbrol\.emulator_console_auth_token
wvcXshnxhRmWXa7S

# Login inside the virtual router and set port forwarding
telnet 127.0.0.1 5554
Android Console: you can find your <auth_token> in
'C:\Users\mbrol\.emulator_console_auth_token'
OK
auth wvcXshnxhRmWXa7S
# For example, the following command sets up a redirection that handles all incoming TCP connections to your host (development) machine on 127.0.0.1:5000 and passes them through to the emulated system on 10.0.2.15:6000
redir add tcp:5000:6000
redir add tcp:10099:27042
PS C:\WINDOWS\system32> netstat -ano | findstr 10099
  TCP    127.0.0.1:10099        0.0.0.0:0              LISTENING       24980

redir list
redir del
```

>[!warning] Warning
>`redir` only binds emulator ports to `localhost`. You need to create another rule on the Host machine to bind `127.0.0.1` to a public port.
>
>Eg. `netsh interface portproxy add v4tov4 listenport=10100 listenaddress=0.0.0.0 connectport=10099 connectaddress=127.0.0.1`
>See [Pivoting](../Web%20&%20Network%20Hacking/Pivoting.md) for further commands and references.
>
>You must also **disable any firewall rule**!

>[!warning] Warning
>To expose the [Frida](Frida.md) server use the `-l` flag! Eg. `./frida-server -l 0.0.0.0:27042`

# External references

- https://gist.github.com/mrk-han/66ac1a724456cadf1c93f4218c6060ae