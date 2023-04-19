# DAST basics
The mobile application is analyzed as it runs on the device. Reviews include forensic analysis of the file system, assessment of the network traffic between the application and server and an assessment of the application’s inter-process communication (IPC).

>[!hint]
>Test the application both online and offline in order to evaluate if it behaves correctly.

>[!warning] 
>Rooted/Jailbroken device are almost mandatory

Main tasks include:
- Network Traffic Analysis
    - Insecure network communication
    - Man in the Middle (MiTM) required
- Backend Analysis
    - VA/PT on API and WebServices (classic Web Application Penetration Test)
- Client-side testing, e.g.
    - Sensitive Data Exposure
        - Inside logs
        - Inside non-censored activities
    - Insecure Configuration
    - Insecure Cryptography
    - Rooting detection

Key security testing areas:
- **Local Data Storage**, e.g. data leak to cloud storage, backups, logs, keyboard cache, db or local files
- **Authentication and Authorization**, e.g. bypassing, insecure or wrongly implemented framework
- **Interaction** with the **Mobile Platform**, e.g. IPC APIs misues, sensitive data or functionality exposure to other apps running on the device.
- **Code Quality** and **Exploit Mitigation** e.g. injection and memory management issues, cross-site scripting
- **Anti-Tampering** and **Anti-Reversing**

# Security mechanism bypasses

## Bypassing Certificate Pinning

>[!warning] 
>**Certificate Pinning**: validation of the certificate given by the back-end on the base of a public key embedded on the application. If Implemented, it blocks the analysis of HTTPS streams via interception proxy: the certificate of the proxy does not match the one embedded in the app.

Evasion techniques:
- **Device Trust Store**: install the proxy certificate into the device trust store (system certificate store or user certificate store based on the Android version and root availability). See [Push burpsuite certificate inside the system certificate store](../Tools/adb.md#Push%20burpsuite%20certificate%20inside%20the%20system%20certificate%20store)
- **App patching**: disassembling the app in order to remove the code responsible for certificate pinning or implement user certificate trust configurations and then [rebuild and sign the «patched» app](../Tools/APKTool.md#Rebuild%20files%20into%20APK)
    - **disassembly tools** + [APKTool](../Tools/APKTool.md)
        - **Smali** code **knowledge** is required
    - **network_security_config.xml +** Add **`android:networkSecurityConfig="@xml/network_security_config"`** to the **`<application>`** element in your application manifest.xml
    - **Apk-mitm**
- **Runtime Manipulation**: runtime execution modification in order to invalidate certificate pinning validation
    - [Xposed!](../Tools/Xposed!.md) framework with [TrustMeAlready](../Tools/Xposed!.md#TrustMeAlready) or [SSLUnpinning - Certificate Pinning Bypass](../Tools/Xposed!.md#SSLUnpinning%20-%20Certificate%20Pinning%20Bypass) (root required)
    - [Frida](../Tools/Frida.md)
    - **Objection** (non-root, see the [following video](https://www.youtube.com/watch?v=qaJBWcueCIA&ab_channel=CorSecure))
- **Improper Cert Management**: the server certificate might be stored in unsafe locations, (Documents directory, application files) therefore replaced with our own proxy certificate

>[!note]
>Android has 3 different certificate store:
>- The “system” one, which is also the default one checked by apps, is located in `/system/etc/security/cacerts/`. This certificate store is pre-populated when the OS is installed the first time and cannot be modified or altered without root privileges.
>- The “user” one, located in `/data/misc/user/0/cacerts-added/`, contains trusted CA certificates manually installed by the user of the device. Those certificates were trusted by default before Android 7, but now a day each application has to specifically opt to trust them.
>- Finally the latest certificate store is the “app itself”. Each application can include its own certificates, refusing to trust any other installed one, even those within the system certificate store. This technique is the so-called “certificate pinning”.



### Bypassing Anti-Root

>[!warning]
> **Anti-Root Mechanism**: the application checks if the device is rooted by looking at different file-system areas. If implemented, it might block a rooted/jailbroken device to run the app or it might hide some sensitive functionalities.

Evasion techniques:
- **App patching**: based on disassembling the app in order to remove the code responsible for root checks and then rebuild the «patched» app
    - **Smali** code **knowledge** is required
    - **disassembly tools** + [APKTool](../Tools/APKTool.md)
- **Runtime Manipulation**: runtime execution modification in order to invalidate root checks
    - **Rooted device** required
    - **Magisk Hide**
    - **Frida**

---

# Tools

## DAST Tools
- [adb](../Tools/adb.md#Dynamic%20analysis)
- Frida
- objection
- [drozer](../Tools/drozer.md)
- [Xposed!](../Tools/Xposed!.md) + [Modules](../Tools/Xposed!.md#Modules)
- [Magisk](https://github.com/topjohnwu/Magisk) (on [Genymotion](https://support.genymotion.com/hc/en-us/articles/360011385178-How-to-install-Xposed-or-Magisk-Edxposed-with-Genymotion-Device-image-PaaS-))
